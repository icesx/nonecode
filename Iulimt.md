
###/etc/security/limits.conf
vi /etc/security/limits.conf

* - nofile 102400
 
###/etc/pam.d/login
session required /lib/security/pam_limits.so 

### /etc/sysctl.cfg

fs.file-max=102400
###/etc/ssh/sshd_config
UsePAM yes
