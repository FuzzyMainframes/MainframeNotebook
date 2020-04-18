# TK4- Tips
## Initial Changes
### Enable Console Mode
To make administration easier, always enable the console mode so that we do not have to use the web console.
```bash
cd unattended
./set_console_mode
cd ..
```
### Update TK4- Configuration to allow dual-CPU
Edit the conf/tk4-.cnf file and update the following values:

```
CPUMODEL 3084
MAXCPU 2
NUMCPU 2
```
