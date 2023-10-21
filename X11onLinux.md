## Opening a Remote GUI using X11 on Linux

1. Based on the website https://help.ubuntu.com/community/ServerGUI, install X11 via the two commands:
 <pre> 
   sudo apt-get install xorg
   sudo apt-get install openbox
 </pre>   

2. <pre> sudo /etc/ssh/ssh_config </pre> and make sure the sure the line <pre> ForwardX11 yes</pre> is commented in.

3. Edit your ssh config file to add X11 forwarding.
   <pre>
     nano ~/.ssh/config
   </pre>
   Then, add the line <pre> ForwardX11 yes </pre> under your ssh header. For me, it looks as follows:
   <pre>
    Host correlator4.fnal.gov
      HostName correlator4.fnal.gov
      LocalForward localhost:8880 localhost:8880
      User rmarroqu
      GSSAPIAuthentication yes
      GSSAPIDelegateCredentials yes
      StrictHostKeyChecking no
      UserKnownHostsFile /dev/null
      Forward X11
   </pre>

4. Not sure if this is needed, but <pre> sudo nano /etc/ssh/ssh_config </pre> and comment in the line <pre> ForwardX11 yes </pre>.

5. After that, ssh into the server. Make sure you are using the local host ssh with the local host that is specified.

6. Now, you should be able to launch a GUI from the terminal on the remote server with correct command. For me, <pre> vivado_hls </pre>
