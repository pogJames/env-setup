# Tera Term
1. USB (Serial)
File -> New Connection -> Serial -> Port: COM
Setup -> Serial Port -> 115200 N81
Login: root, root
2. Ethernet (TCP/IP)
File -> New Connection -> TCP/IP -> Host: check using `ifconfig`, Service: SSH

### How to put iess302 folder to Matrix 752
1. compress file to .tar
2. PC commmand prompt: $scp iess302.tar root@`752TCP/IP`:/tmp/
3. Tera term: $tar -xvf iess302.tar -C /home/root
4. 
