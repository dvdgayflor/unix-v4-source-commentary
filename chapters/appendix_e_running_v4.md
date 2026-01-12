# Appendix E: Running UNIX v4

This appendix explains how to run UNIX v4 on a modern computer using the OpenSIMH PDP-11 simulator. Running the actual system lets you experiment with the code discussed throughout this book.

## Resources

The UNIX v4 restoration and emulation documentation is maintained at:

- **squoze.net** — [http://squoze.net/UNIX/v4/](http://squoze.net/UNIX/v4/) — Complete restoration by Angelo Papenhoff
- **Internet Archive** — [https://archive.org/details/utah_unix_v4_raw](https://archive.org/details/utah_unix_v4_raw) — Original tape image
- **Turnkey version** — [http://squoze.net/UNIX/v4/turnkey/](http://squoze.net/UNIX/v4/turnkey/) — Pre-built, ready to boot

For the most up-to-date instructions, consult squoze.net. The instructions below provide a quick start.

## Prerequisites

### macOS (MacPorts)

```bash
sudo port install opensimh
```

OpenSIMH executables have a `simh-` prefix (e.g., `simh-pdp11`).

### Debian/Ubuntu

```bash
sudo apt install simh
```

The executable is named `pdp11`.

## Quick Start with Turnkey Version

The fastest way to get running:

```bash
# Download pre-built disk and boot script
curl -O http://squoze.net/UNIX/v4/turnkey/disk.rk
curl -O http://squoze.net/UNIX/v4/turnkey/boot.ini

# Boot UNIX v4
simh-pdp11 boot.ini
```

At the boot prompt:

```
k
unix
```

Login as `root` (no password).

## Installation from Tape

For the full installation experience:

### Step 1: Download Files

```bash
curl -O http://squoze.net/UNIX/v4/unix_v4.tap
curl -O http://squoze.net/UNIX/v4/install.ini
curl -O http://squoze.net/UNIX/v4/boot.ini
```

### Step 2: Run Installation

```bash
simh-pdp11 install.ini
```

At the `=` prompt, enter:

```
=mcopy
'p' for rp; 'k' for rk
k
disk offset
0
tape offset
75
count
4000
=uboot
k
unix
```

You should see:

```
mem = 64530

login: root
```

### Step 3: Verify Installation

```bash
# ls
bin
dev
etc
lib
mnt
tmp
unix
usr
# sync
# sync
```

## Basic Usage

### Important: Use `chdir` not `cd`

UNIX v4 predates the `cd` alias. You must use `chdir` to change directories:

```bash
# chdir /usr      # correct
# cd /usr         # wrong - no such command
```

### Essential Commands

```bash
# ls              # list files
# ls -l           # long listing
# pwd             # print working directory
# chdir /usr      # change directory
# cat file        # display file
# ps              # show processes
# who             # show logged-in users
# date            # show date (will show 1974!)
# df              # disk free space
```

### Text Editing with ed

The only editor is `ed`, a line editor:

```bash
# ed filename
a                 # append mode
Type your text here
.                 # period alone exits append mode
w                 # write (save)
q                 # quit
```

### Compiling C Programs

```bash
# ed hello.c
a
main()
{
    printf("Hello from UNIX v4!\n");
}
.
w
q
# cc hello.c      # compile
# a.out           # run
Hello from UNIX v4!
```

## Shutdown Procedure

**Critical:** Always follow this procedure to prevent data loss.

```bash
# sync            # flush buffers
# sync            # run twice to ensure completion
```

Then press `Ctrl+E` to enter the simulator prompt:

```
Simulation stopped, PC: 002040 (MOV (SP)+,177776)
sim> quit
Goodbye
```

**Why sync twice?** The first `sync` starts the buffer flush but doesn't guarantee completion. The second ensures all data is written.

## Optional: Rebuilding the Kernel

If `ps` doesn't work or you want to customize the kernel:

### Step 1: Download Kernel Build Files

```bash
curl -O http://squoze.net/UNIX/v4/sys.tp
```

### Step 2: Update boot.ini

```
set cpu 11/45
set tc en
att rk0 disk.rk
att tc1 sys.tp
d sr 2
boot rk
```

### Step 3: Extract and Build

```bash
# chdir /usr/sys
# tp 1x           # extract from DECtape
# sh run          # build kernel
# mv a.out /nunix
# sync
# sync
```

### Step 4: Boot New Kernel

Restart the simulator and at the boot prompt:

```
k
nunix            # boot new kernel instead of "unix"
```

If successful, replace the old kernel:

```bash
# mv /unix /unix.old
# mv /nunix /unix
```

## Optional: Installing Manual Pages

UNIX v4 didn't include manual pages. The restoration provides them from v6:

```bash
curl -O http://squoze.net/UNIX/v4/nroff.tp
curl -O http://squoze.net/UNIX/v4/man.tap
```

See squoze.net for detailed installation instructions. Note that `nroff` has a known bug where it resets the terminal to uppercase mode—use output redirection (`man cat > temp && cat temp`) to work around this.

## Troubleshooting

**Stuck at `=` prompt:** Type `uboot` then `k` then `unix`

**Lost data after reboot:** Always run `sync` twice before exiting

**ps command doesn't work:** Rebuild the kernel with the updated files from sys.tp

**Terminal stuck in UPPERCASE:** Run `stty -lcase` or logout and login again

## Experiments to Try

With a running UNIX v4 system, you can verify the code discussed in this book:

1. **Examine the filesystem:**

   ```bash
   # ls -l /dev
   # cat /etc/passwd
   # od /unix | head
   ```

2. **Watch process creation:**

   ```bash
   # ps a
   # sh &
   # ps a
   ```

3. **Explore the C compiler:**

   ```bash
   # chdir /usr/c
   # ls
   # cat c00.c | head -50
   ```

4. **Look at the kernel source:**

   ```bash
   # chdir /usr/sys/ken
   # ls
   # cat main.c
   ```

5. **Try the assembler:**

   ```bash
   # chdir /usr/source/s1
   # ls *.s
   # cat cat.s
   ```

## File Descriptions

| File | Description |
|------|-------------|
| unix_v4.tap | Original tape in SIMH format |
| disk.rk | RK05 disk image (your filesystem) |
| install.ini | SIMH config for installation |
| boot.ini | SIMH config for booting |
| sys.tp | DECtape with kernel build files |
| nroff.tp | Text formatting system from v6 |
| man.tap | Manual pages |

## Notes

- UNIX v4 has no password for root by default
- The system uses 64KB of memory
- RK05 disk images are approximately 2.4MB
- Press `Ctrl+E` at any time for the SIMH prompt
- The date will show 1974—this is correct for the tape

---

For comprehensive documentation and the latest updates, visit [squoze.net/UNIX/v4](http://squoze.net/UNIX/v4/).
