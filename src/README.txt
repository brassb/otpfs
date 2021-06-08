One-Time-Pad Filesystem (otpfs)

Implemented using Linux FUSE (Filesystem in User Space)

Written by Bill Brassfield ( https://github.com/brassb )

June 2021



To build (on Linux):

1. Be sure 'gcc' and 'make' are installed on the system.

2. Make sure your Linux system supports FUSE and has the
   C header files and libraries.

3. Make sure the OpenSSL development C header files and
   libraries are installed.

4. In this src directory, run:

   make



If all went well, you should have a new executable:

  otpfs

Before running it, make sure you have a raw storage
directory (for your encrypted files) and a mountpoint
directory (where this FUSE filesytem will be mounted).

Examples of commands to run and mount otpfs:

  ./otpfs -f --user_data source_dir=/home/johndoe/Raw_Data /home/johndoe/OTPFS_Mount_Point

  ./otpfs -f --user_data show_passphrase=yes,source_dir=/home/johndoe/Raw_Data /home/johndoe/OTPFS_Mount_Point

  ./otpfs -f --user_data show_passphrase=yes,serial_start=5,serial_step=17,source_dir=/home/johndoe/Raw_Data /home/johndoe/OTPFS_Mount_Point

  ./otpfs -f --user_data show_passphrase=yes,serial_type=int,serial_start=5,serial_step=17,source_dir=/home/johndoe/Raw_Data /home/johndoe/OTPFS_Mount_Point

  ./otpfs -f --user_data show_passphrase=yes,serial_type=long,serial_start=5,serial_step=17,source_dir=/home/johndoe/Raw_Data /home/johndoe/OTPFS_Mount_Point

  ./otpfs -f --user_data show_passphrase=no,serial_type=long,serial_start=27182818284,serial_step=314159,source_dir=/home/johndoe/Raw_Data /home/johndoe/OTPFS_Mount_Point

(NOTE: You may want to run this command inside a 'screen' or 'tmux'
session.  It requires a terminal because it occasionally outputs
messages to standard output.)



The result: Files which are created and modified under OTPFS_Mount_Point
will appear as 'plaintext' in this directory, while the same files will
be ciphertext (XOR'ed with the pseudo-random one-time-pad) in the Raw_Data
directory.  After the otpfs filesystem has been unmounted (by quitting the
otpfs executable with CTRL-C or a SIGTERM kill signal), only the ciphertext
files will be visible under Raw_Data.  In order to access the plaintext
data, it will be necessary to run otpfs again, and pass it exactly the same
passphrase, serial_type, serial_start, and serial_step values.  Even a
one-bit change in any of these will leave the data unrecoverable.

If the program crashes for any reason, the mount-point directory may look
strange with its "transport endpoint disconnected".  In order to fix this,
simply run something like:

  fusermount -u /home/johndoe/OTPFS_Mount_Point

and it will clear this up, making the mount point directory look normal
again.

Plain files of any type (7-bit ASCII text files, or 8-bit 'binary' files)
can be encrypted/decrypted with otpfs.  One interesting thing to do is to
edit a plaintext file under OTPFS_Mount_Point and make a simple change to
it -- then use an editor with a binary hex-edit mode to see what got changed
in the underlying ciphertext file under Raw_Data.  You will note that only
the characters (or bytes in the file) which changed in the plaintext file
will also change in the ciphertext file.  This is because this encryption
is NOT a block cipher.  It's a simple bit-stream cipher, and the pseudo-
random bit-stream used for the XOR-based encryption/decryption is really
intended to be used only ONCE (i.e., for only ONE plaintext).

One type of file which is less vulnerable to such an attack is a
LUKS-encrypted disk image file (formatted using 'cryptsetup luksFormat', and
opened with 'cryptsetup luksOpen').  If such a disk image is LUKS-readable
under the OTPFS_Mount_Point, then any changes in the disk image's filesystem
will cause entire blocks of data in the disk image to change because LUKS uses
a block-cipher such as AES-128 or AES-256.  This means entire blocks of bits
will also be changed in the underlying Raw_Data otpfs-ciphertext version of
the disk-image file.  The LUKS headers in the first 1MB of the disk image
file should be unreadable in the Raw_Data version of the file.  Only after
a successful mount of otpfs with the correct passphrase, serial_type,
serial_start, and serial_step can the 'cryptsetup isLuks' and 'cryptsetup
luksOpen' commands make any sense out of the disk image file under the
OTPFS_Mount_point.



DISCLAIMER:

This software is provided free and open-source, as-is, and comes with no
warranty or assumption of any liability by the author(s) of the software.
Read the GNU GPL (General Public License) for details on how this works.

This is NOT production software.  DO NOT, under any circumstances, rely on
this software to provide you with any form of data security and/or data
integrity.  The encryption used in this software is pseudo-randomly-generated
"one-time-pad" bits XOR'ed with the data to transform between plaintext and
ciphertext.  Incorrectly using this type of encryption (such as encrypting
more than one plaintext using the same stream of one-time-pad bits) may
expose your secrets to cryptanalysis attacks.

Furthermore, it's quite possible that this software could lead to permanent
data corruption and/or total loss of data, due to software bugs and/or loss
of key pieces of data used in generating the pseudo-one-time-pad stream of
bits:  These include the pass phrase (which is both case and white-space
sensitive), the serial-numbering data type (int or long), the starting
serial number, and the serial-numbering increment size.  The serial
numbering scheme defaults these settings to (int, 0, 1), but these may be
easily overridden in the name-value pairs passed into the user_data argument.

This software is for teaching, learning, and experimental computer-lab-use
ONLY.  This software is the equivalant of a child's toy, made out of easy-
to-break plastic, and just strong enough to survive 5 minutes after the birthday
gift is unwrapped before it breaks.  This is not heavy-duty, super-robust,
enterprise-grade 6-figure software made out of bullet-proof Uranium tank
armor.  It may segfault and core-dump, it may crash your Linux system, or it
may irrecoverably trash your precious data with which you have entrusted it.

YOU HAVE BEEN WARNED.  USE THIS SOFTWARE AT YOUR OWN RISK.

But at the same time, please have fun with it.  And hopefully, you will
learn something and be inspired to write your own Linux FUSE filesystem
to do something immensely more useful.
