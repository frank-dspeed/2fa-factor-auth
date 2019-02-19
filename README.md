# 2fa-factor-auth
Lining out 2fA




apt-get install libpam-google-authenticator





In /etc/pam.d/sshd I have changed/added the following lines (at the top):

# @include common-auth
# auth sufficient pam_google_authenticator.so
auth required pam_google_authenticator.so
And in /etc/ssh/sshd_config:

ChallengeResponseAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive
PasswordAuthentication no




I was finally able to get this working by placing auth [success=done new_authtok_reqd=done default=die] pam_google_authenticator.so nullok at the top of /etc/pam.d/sshd.

According to the pam.d man page:

success=done means that if Google Authenticator signs off, no more authentication will be performed, meaning no additional password prompt.
default=die means that if Google Authenticator rejects the login attempt, authentication will immediately fail, skipping the password prompt.
So [success=done new_authtok_reqd=done default=die] is sort of a mix between the sufficient and requisite control values, since we want behavior from both: if success, terminate immediately (sufficient), and if failure, also terminate immediately (requisite).

Note that the nullok argument to pam_google_authenticator.so means that if a ~/.google_authenticator file is not found for a user, public-key authentication proceeds as normal. This is useful if I want to lock down only a subset of my accounts with 2FA.

As a side note, if you wish to have an sftp user account, you will probably need to bypass google authenticator in order to get it to work. Here is a suggestion of how to do that securely using an sftp jail. In etc/ssh/sshd_config:

Subsystem sftp internal-sftp
Match User ftp-user
  PasswordAuthentication yes
  AuthenticationMethods password
  ChrootDirectory /path/to/ftp/dir
  ForceCommand internal-sftp
You will need to make the permissions on /path/to/ftp/dir root write only (e.g. chown root:root /path/to/ftp/dir, chmod 755 /path/to/ftp/dir. All of the parents above that directory also need secure permissions. The way I usually do this is by making the chroot directory /home/shared/user, creating a directory in there (e.g. 'data') and then mounting whatever directory I want to share like this: sudo mount -o bind /path/to/ftp/dir /home/shared/user/data

If you follow all of those steps, you will have public key + google authenticator login for your ssh users, and a functional password protected sftp account for data transfer.


