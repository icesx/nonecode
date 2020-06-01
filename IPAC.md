PAC在ubuntu17.04上的有问题，无法启动，经查可能是perl版本的问题，解决办法如下
```
	#wget http://www.cpan.org/authors/id/X/XA/XAOC/Gnome2-Vte-0.11.tar.gz
	#tar -xzvf Gnome2-Vte-0.11.tar.gz
	#cd Gnome2-Vte-0.11
	#sudo apt-get install libvte-dev dh-make-perl
	#sudo apt-get install apt-file
	#sudo apt-file update	
	#dh-make-perl --cpan Gnome2::Vte --build
	#sudo dpkg -i libgnome2-vte-perl_0.11-1_amd64.deb
	#cd /opt/pac/
	#find . -name "Vte.so*" -exec rm {} \;
	#sudo dpkg -k pac..deb
```