Cats - 450
=========
<h4>Problem:</h4>

	I had fun once, it was horrible.
	Password: F3lyn34LifE!
	cats_51474229fe0d9bbd8da500fe3f74b383d175cee1.zip


<h4>Link:</h4>
 https://d34j1dfl6n327i.cloudfront.net/cats_51474229fe0d9bbd8da500fe3f74b383d175cee1.zip

----

<h4>Video Walkthrough Link:</h4>
https://www.youtube.com/watch?v=zoWxoGbVd8Q

----
<h4>Solution:</h4>

First you download the file. As always, make sure you take the SHA-1 checksum of the file before doing anything and see if it matches up with the filename, to make sure you downloaded the right file and not a virus or something.

In this challenge, we are given a virtual machine of someone who evidently really likes cats. Of course, we will not log in to the VM - this is a no-no in forensics for multiple reasons:

 - We lose forensic integrity
 - Temporary files may be lost 
 - Memory locations get overwritten (deleted files, etc)
 - Unwanted programs may run, etc.

Actually, we shouldn't even boot from the VM's hard disk. Instead, we use it as an unmounted hard disk in Kali. But first, take a snapshot of the VM, just in case you accidentally make some mistakes (and so that we don't lose any extra-volatile data, in case those are necessary). In order to do all these things, we:

1. Go over to our virtualization software (hypervisor) of choice. In this case, we used VirtualBox, but you could use VMWare or Parallels too. (This is where the snapshot can be taken.)
2. In the CD/DVD slot, we insert a Kali Linux ISO. Find one at https://www.kali.org/downloads/.
3. When prompted, boot into forensics mode. Forensics mode makes sure nothing ever happens to the hard drive without direct user interaction (http://docs.kali.org/general-use/kali-linux-forensics-mode).
![Kali Bootup](https://www.dropbox.com/s/lteovojvqu754ew/Screen%20Shot%20on%202015-10-09%20at%2022-14-17.png?raw=1)
4. In Kali, we first get a SHA-1 checksum of the partition, which we will check later again to make sure nothing changed. We run `openssl sha1 /dev/sda1`, and get back `ccef3f6b74e943d0e020de56c992bccd21de09af`.
5. Mark the virtual hard drive as read only: `blockdev --setro /dev/sda`
6. We now mount the hard drive:
`mkdir /mnt/cats`, then `mount -r /dev/sda1 /mnt/cats`
(http://askubuntu.com/questions/20680/what-does-it-mean-to-mount-something). As long as we don't add or change any files on the hard drive, we maintain forensic integrity.
![Mounting the drive](https://www.dropbox.com/s/zsc5u2bqgocqda9/Screenshot%20from%202015-10-10%2023%3A35%3A14.png?raw=1)

---
Now, onto actually looking through this VM for a flag.

First, we look into the user's bash history (`.bash_history` or the backup file `.bash_history~`, in `/mnt/cats/home/catz/`) to see if they did anything in the terminal recently. We do this because there is often useful information in the bash_history, and it will likely point us in the right direction on how to solve this problem. In the `.bash_history` file, we only see `clear`, but in the backup of the bash history (`.bash_history~`) we get:
![Bash History](https://www.dropbox.com/s/8s3wgysnthqu4cj/Screenshot%20from%202015-10-10%2022%3A38%3A07.png?raw=1)
It seems that there is some volume that has been encrypted by VeraCrypt (https://veracrypt.codeplex.com/)! This volume is conveniently located at `/home/cats/catz`, or in our case, `/mnt/cats/home/cats/catz` since we mounted the drive there.

In addition, we see many mentions of `logkeys.log`. What could it be? Let's take a look at that log file, in case it's relevant. We run `nano /mnt/cats/var/log/logkeys.log` and we see:
![logkeys.log dump](https://www.dropbox.com/s/6cc9fplhe6vig48/Screenshot%20from%202015-10-10%2022%3A39%3A33.png?raw=1)

From this, it seems that there was a keylogger running on the computer that logged everything that the user typed. This gives us the passphrase to decrypt the VeraCrypt (`Me0wL3tMeInPl$`)!

(Note: this keylogger seems to have somehow logged its own installation. lol)

In order to decrypt the VeraCrypt in our Kali LiveCD, we have to first install VeraCrypt. Note: we aren't installing VeraCrypt on the hard disk, only on the Kali LiveCD that we're running from, so we maintain forensic integrity. We want the same version of the software that was used to create the encrypted volume, so that changes made in the newer versions will not affect the output. In order to find the version of VeraCrypt, we `grep -r "veracrypt"`, and we see that they used `veracrypt-1.0e`.
![Grep of Veracrypt](https://www.dropbox.com/s/68yzr2ollx209au/Screenshot%20from%202015-10-10%2022:41:07.png?raw=1)

So, we just fire up a browser and install `veracrypt-1.0e`.
![Downloading VeraCrypt](https://www.dropbox.com/s/l5vmnzm54w81q8o/Screenshot%20from%202015-10-10%2022:43:36.png?raw=1)
After downloading the Linux version, we install it by unzipping the file, going to the file location (`~/Downloads`) and then running:
`./veracrypt-1.0e-setup-console-x64`, selecting install, and accepting the terms and conditions.
![Setup](https://www.dropbox.com/s/p5i796ax5jpcren/Screenshot%20from%202015-10-10%2022:53:21.png?raw=1)
After installing, we just use VeraCrypt to decrypt the encrypted `catz` file!

All we need to do is copy pretty much everything we find in the `logkeys.log` file. After a while, VeraCrypt successfully decrypted the file and mounted it. What we typed was:

1. First make the mount point. `mkdir /mnt/catzmntpt`
2. Decrypt and mount the file: `veracrypt -t -k "" --protect-hidden=no catz /mnt/catzmntpt`
![Decrypting catz](https://www.dropbox.com/s/t3ux14i0xjrehel/Screenshot%20from%202015-10-10%2022:57:15.png?raw=1)
3. Password: `Me0wL3tMeInPl$`

In the newly mounted drive, we see a CATZZZ folder, which has lots of files.
![CATZZZ](https://www.dropbox.com/s/d46rpsis9q7uan7/Screenshot%20from%202015-10-10%2023:04:52.png?raw=1)
In file viewer, we see that many of these files have pictures and/or text in them. However, the thumbnail of `grooming.pdf` is blank.
![Thumbnails](https://www.dropbox.com/s/906nqohsydpd7u9/Screenshot%20from%202015-10-10%2023%3A05%3A22.png?raw=1)
That seems pretty suspicious. So, we open it, and select all.
![Grooming.pdf](https://www.dropbox.com/s/l2k108lmez3pb86/Screenshot%20from%202015-10-10%2023%3A05%3A59.png?raw=1)

There's the flag!
`{DX5wMM0pCiNwtxf5bEhNscglFQOWHjNECrp2NYBR}`

If this were a real forensic investigation, though, it wouldn't be over yet. Now we need to show that we made no changes to the hard disk. To do this, we just hash `/dev/sda1` again (`openssl sha1 /dev/sda1`), and out we get:
`ccef3f6b74e943d0e020de56c992bccd21de09af`
![rehashed](https://www.dropbox.com/s/eedau2a6dw8a4bg/Screenshot%20from%202015-10-11%2000:00:06.png?raw=1)
It matches with our first hash! So we have maintained forensic integrity while getting ourselves 450 points.

-------
> Writeup by PHS Absol.
> 
> Work Division:
> Downloaded the VM and found stuff on it, made the video - Plato2000
> Gave forensic integrity tips, wrote the writeup - Neptunia
> Found the flag in the decrypted files - Flareboot
