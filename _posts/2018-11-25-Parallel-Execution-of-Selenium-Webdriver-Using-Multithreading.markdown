---
layout:     post
title:      "Road to Parallel Execution of Selenium Webdriver Using Multithreading"
date:       2018-11-25
author:     "Tech"
header-img: "img/post-selenium.png"
tags:
    - Programming
---

# Road to Parallel Execution of Selenium Webdriver Using Multithreading :
Some highlights of selenium:
  - Automates browsers
  - Can be run in headless mode if needed
  - Since PhantomJS is outdated, Selenium is the new replacement.

Somethings not so nice about selenium:
  - Was not built for multi-threading.
  - Not many projects written in ruby, which makes research a little bit more intensive.
  - Selenium likes to throw errors, sometimes for no reason.

This project came about after using Kali's Eyewitness on a decently size internal network. Eyewitness isn't the best software to use on a large network (500k+), to many false positives and dreadful *miscommunication* results lead to the creation of this project.

### Setting up:

You'll be running '**./Autorun.sh**' to start the script, in that script file there are a few parameters you'll need to set. These parameters are located in the beginning of the script.

These parameters are explained below:

* **NetworkAdapter**- Set this to your currently used network adapter, this is used to make sure there is a connection when running.
* **List**- This csv file would be the output of this script: nmap_xml2csvOpenWeb_Socket.py
* **Outputfile**- This is the output database file for the titlegrabber script.
* **LogsFile**- This is a general logger to keep track of errors.
* **Script_LogsFile**- This is another general terminal logger to keep track.
* **Set_Exit_Time**- This is used if you want to set a time-timeout. ie: 11pm shut off script. By default I turned this off.

TitleGrabber will make a few files, Database.csv (the default name of the outputfile parameter), New_Monthly.csv (the default name of the List parameter), the two logs files, and New_Monthly.csv_SPLIT which is used as a counting down metric and keeping track.

When TitleGrabber finishes, your left with the Database.csv. From here you'll now move ontop using TitleParser...


Besure to have all the required gems installed:
```bash
gem install date optparse rubygems resolv socket timeout rex thread net/http uri selenium-webdriver time pry

https://chromedriver.storage.googleapis.com/2.35/chromedriver_linux64.zip
gem install selenium-webdriver
cp chromedriver /usr/bin/
apt-get install chromium
apt-get install libgconf-2-4
gem install headless
sudo apt-get install xvfb
```


### TitleGrabber.rb
```ruby
You will most likely fine the most updated version of this project directly on my github repo, please pull from there. For quick reference I have attached the code below:
The script is located at my github: [https://github.com/Techryptic/Parallel-Execution-of-Selenium-Webdriver-Using-Multithreading](https://github.com/Techryptic/Parallel-Execution-of-Selenium-Webdriver-Using-Multithreading)

#!/usr/bin/env ruby
require 'date'
require 'optparse'
require 'rubygems'
require 'resolv'
require 'socket'
require 'timeout'
require 'rex'
require 'thread'
require 'net/http'
require 'uri'
require 'selenium-webdriver'
require 'time'
require 'pry'
@write_mutex = Mutex.new
STDOUT.sync = true

def CheckFile(csv,realcsv=nil)
    csv = csv.strip
    ip = csv.split(',')[9]
    address = ip.strip.chomp

    if File.readlines("Database.csv").grep(/#{address}/).size > 0
        puts 'Skipping dups'
    else

    if csv.include? "F5 BIG-IP load balancer"
        File.open('bannergrabber.csv', 'a') do |f|
	         f.write "#{csv.strip},F5 BIG-IP load balancer\n"
             @write_mutex.synchronize{puts "Wrote to file, F5 BIG-IP load balancer------------------------------"}
        end
    else
        realcsv = address
        return realcsv
    end
    end
end

def InitialRequestBOTH(csv, realcsv)
    address = realcsv.strip
    capabilities = Selenium::WebDriver::Remote::Capabilities.firefox(accept_insecure_certs: true, acceptSslCerts: true, handlesAlerts: true, unexpectedAlertBehaviour: true)
    driver = Selenium::WebDriver.for :firefox, :desired_capabilities => capabilities
    driver.manage.timeouts.page_load = 20
	 begin
		Timeout::timeout(21) do
			#driver.manage.window.resize_to(1000, 1000)# resize the window and take a screenshot
			#driver.save_screenshot "thread.png"
			url = "#{address.chomp.strip}"
         protocol = url.split("\/").first
         domain = url.split("\/").last
         url = protocol+"//admin:admin@"+domain+"/"
			driver.navigate.to(url)
         sleep(3)

			if driver.title.empty?
				File.open('bannergrabber.csv', 'a') do |f|
					 f.write "#{csv.strip},NO TITLE\n"
                     @write_mutex.synchronize{puts "Wrote to file, No Title------------------------------"}
			    end
                driver.quit
			else

            if driver.title.include? "401 Unauthorized" and driver.current_url.include? "/view/view.shtml?id="
				File.open('bannergrabber.csv', 'a') do |f|
					 f.write "#{csv.strip},Axis Camera\n"
                     @write_mutex.synchronize{puts "Wrote to file, Axis Camera------------------------------"}
			    end
                driver.quit
            else

            time1 = Time.new
			@write_mutex.synchronize{puts "#{url},#{driver.title}\t#{time1.inspect}"}
				File.open('bannergrabber.csv', 'a') do |f|
					 f.write "#{csv.strip},#{driver.title}\n"
                     @write_mutex.synchronize{puts "Wrote to file, now to call driver.quit------------------------------"}
                     driver.quit
			end
			end
         end
		end
        rescue Selenium::WebDriver::Error::UnexpectedAlertOpenError
		File.open('bannergrabber.csv', 'a') do |f|
			 f.write "#{csv.strip},Error::UnexpectedAlertOpenError\n"
             @write_mutex.synchronize{puts "Error::UnexpectedAlertOpenError-PRINTED TO FILE------------------------------"}
	    end
        driver.quit

        rescue Selenium::WebDriver::Error::UnhandledAlertError
		File.open('bannergrabber.csv', 'a') do |f|
			 f.write "#{csv.strip},Error::UnhandledAlertError\n"
             @write_mutex.synchronize{puts "Error::UnhandledAlertError-PRINTED TO FILE------------------------------"}
	    end
        driver.quit

        rescue Net::ReadTimeout
        @write_mutex.synchronize{puts "Net::ReadTimeout------------------------------"}
        driver.close

        rescue Timeout::Error
		File.open('bannergrabber.csv', 'a') do |f|
			 f.write "#{csv.strip},Timeout::Error\n"
             @write_mutex.synchronize{puts "Timeout::Error-PRINTED TO FILE------------------------------"}
	    end
        driver.quit

        rescue Selenium::WebDriver::Error::TimeOutError
		File.open('bannergrabber.csv', 'a') do |f|
			 f.write "#{csv.strip},Error::TimeOutError\n"
             @write_mutex.synchronize{puts "Error::TimeOutError-PRINTED TO FILE------------------------------"}
	    end
        driver.quit

        rescue Selenium::WebDriver::Error::NoSuchDriverError
        @write_mutex.synchronize{puts "Selenium::WebDriver::Error::NoSuchDriverError------------------------------"}
        driver.quit

        rescue Errno::ECONNREFUSED
		File.open('bannergrabber.csv', 'a') do |f|
			 f.write "#{csv.strip},Errno::ECONNREFUSED\n"
             @write_mutex.synchronize{puts "Errno::ECONNREFUSED-PRINTED TO FILE------------------------------"}
	    end
        driver.quit

        rescue EOFError
        @write_mutex.synchronize{puts "EOFError------------------------------"}
        driver.quit

		rescue Exception => ex
        @write_mutex.synchronize{puts "Exception------------------------------"}
		$stderr.puts File.expand_path $0
		$stderr.puts ex.class
		$stderr.puts ex
        driver.quit
	end
end

options={}
optparse = OptionParser.new do |opts|
	opts.banner = "Usage: TitleGrabber.rb [options] ip/file"
	options[:list] = false
	opts.on( '-l', '--list', 'Take input from a text file list, one IP per line') do
		$stderr.print "Reading input file..."
		list = File.readlines(ARGV[0])
		$stderr.print "...file read.\nCreating Mutex and threads..."
		THREAD_COUNT = 10
		mutex = Mutex.new
		$stderr.print "...mutex created. Start! #{File.expand_path $0}\n"
		THREAD_COUNT.times.map{
			Thread.new(list) do |ips|
				while ip = mutex.synchronize {ips.pop}
					initValue = CheckFile(ip.gsub(/\n/, ""))
                    if initValue
					    InitialRequestBOTH(ip.gsub(/\n/, ""), initValue)
                    end
				end
			end
		}.each(&:join)
	end
	options[:single] = false
	opts.on( '-s', '--single', 'Take input of a single IP passed through the command line') do
		$stderr.flush
		initValue = CheckFile(ARGV[0])
        if initValue
		    InitialRequestBOTH(ARGV[0], initValue)
        end
		exit
	end
end.parse!
```

For production environments...

```sh
$ npm install --production
$ NODE_ENV=production node app
```

---

# Autorun . sh
I made an Autorun. sh as it fixes some of the overall issues with Selenium crashing. If Selenium does crash, it will start back right where it left off, which is great if your working on a large network.

You will most likely fine the most updated version of this project directly on my github repo, please pull from there. For quick reference I have attached the code below:
The script is located at my github: [https://github.com/Techryptic/Parallel-Execution-of-Selenium-Webdriver-Using-Multithreading](https://github.com/Techryptic/Parallel-Execution-of-Selenium-Webdriver-Using-Multithreading)
```bash
#! /bin/bash

NetworkAdapter="eno1"
List="Nmap-InputFile"
OutputFile="bannergrabber.csv"
LogsFile="logs.txt"
Script_LogsFile="script_logs.txt"
Set_Exit_Time="no"

echo "" > $Script_LogsFile
echo "" > $LogsFile

#BURP
#gnome-terminal -e 'sh -c  "echo STARTING BURP HEADLESS; java -jar -Xmx1g -Djava.awt.headless=true /usr/local/BurpSuitePro/burpsuite_pro.jar --project-file='$BurpProjectFile'"'

#Logger
gnome-terminal -e 'sh -c  "echo STARTING LOGGER; less +F '$Script_LogsFile'"'

CurrentTime=$(date | awk '{print $4}' | cut -d ':' -f1)
OriSizeOutputFileSize=$(cat $OutputFile | wc -l)
ExitHour=23
SECONDS=0
sleep 5

Cyan='\033[1;36m'
Red='\033[1;31m'
Green='\033[1;32m'
none='\e[0m'
#echo -e ell ${Cyan} hello ${none}

while sleep 15;
do echo "" &&
if [ $Set_Exit_Time == yes ]; then
#if (( $CurrentTime >= $ExitHour )); then
    echo "EXECUTION TIME"
    echo "EXITING, Logger, Script.....................$(date)"
    echo "EXITING, Logger, Script.....................$(date)" >> $LogsFile
    pkill less
    pkill ruby
    exit
else
    echo "---------------------------------"
    echo "Currently Running: TitleGrabber.rb"
    Scriptlogs=$(cat $Script_LogsFile | wc -l)
    ListSize=$(cat $List | wc -l)
    OutputFileSize=$(cat $OutputFile | wc -l)

    #Lastmachine Details
    lastbox=$(tail -1 $OutputFile | head -1 | cut -d ',' -f10) #http://ip:port #tail -1 bannergrabber.csv | head -1 | cut-d '.' -f10

    areyouhere=$(grep -n $lastbox $List >> /dev/null 2>&1 && echo yes || echo no)
    if [ $areyouhere == no ]; then
        lastboxfull=$(tail -1 $OutputFile)
        echo $lastboxfull >> $List
    fi

    whichline=$(grep -n $lastbox $List | head -n 1 | cut -d: -f1) #grep -n http://ip:port nmap-output.csv | head -n 1 | cut -d: -f1
    newline=$(jq -n $whichline+0)
    ListSplit=$List"_SPLIT"
    NewListFile=$(head -n $newline $List  > $ListSplit)
    CurrentlyLeft=$(wc -l < $ListSplit)

    #Real Percent based off script_log.txt
    #echo -e "Completed: "${Cyan}$Scriptlogs${none}, out of ${Red}$ListSize${none}

    #Real Calc based off completed:
    RealNumb=$(jq -n $ListSize-$CurrentlyLeft)
    echo -e "Completed: "${Cyan}$RealNumb${none}, out of ${Red}$ListSize${none}

    #Dec=$(echo 5k $Scriptlogs $ListSize /p | dc)
    RealPerc=$(jq -n $RealNumb/$ListSize)
    Percent=$(jq -n $RealPerc*100)
    echo -e "Percent Completed: "${Cyan}$Percent"%"${none}

    duration=$SECONDS
    echo -e "Completed ${Cyan}$Scriptlogs${none} in $(($duration / 3600)):$(($duration / 60)):$(($duration % 60))"

    #Threads (Beta)
    Threads=$(xwininfo -tree -root | grep "Chromium" | awk '{print $1}' | xargs -n1 xwininfo -all -id | grep "Process id" | sort | uniq -c | cut -d "P" -f 1 | xargs)
    T=$(jq -n $Threads-2)
    echo -e "Current Threads Running: "${Cyan}$T${none}

    #Last Machine Worked on (fixing long skip dups)
    echo -e "Last Box: ${Red}$lastbox${none}, Split List: ${Cyan}$ListSplit${none}, Currently Left: ${Cyan}$CurrentlyLeft${none}"

    currentuptime=$(uptime)
    echo Uptime:$currentuptime

    CurrentTime=$(date | awk '{print $4}' | cut -d ':' -f1)
    let countdown=$ExitHour-$CurrentTime
    echo "Set to EXIT in "$countdown" hours!"

    RESULTS=$(ps ax | grep -v grep | grep ruby | wc -l)
    if [ $RESULTS == 2 ]; then
        echo -e ${Green}"Running Great...."${none}
        echo "---------------------------------"
        echo ""
        echo "Running Great... $(date)" >> $LogsFile
    fi

    if [ $RESULTS == 0 ]; then
        echo -e ${Red}"Restarting TitleGrabber.rb..."${none}
        echo "---------------------------------"
        echo "Checking if completed.."

        #CheckScriptLogs=$(cat $Script_LogsFile | wc -l)
        if (( $CurrentlyLeft <= 5 )); then
            echo "Completed: ALL FINISHED"
            pkill less
            pkill ruby
            exit
        fi

        echo "Continuing.."
        echo "Restarting TitleGrabber.rb...: $(date)" >> $LogsFile
        echo "" > $Script_LogsFile
        pkill less
        gnome-terminal -e 'sh -c  "echo STARTING LOGGER; less +F '$Script_LogsFile'"'

        #Split or no
        ToSplit=$(ls $ListSplit >> /dev/null 2>&1 && echo yes || echo no)
        echo $ToSplit
        if [ $ToSplit == yes ]; then
            gnome-terminal --geometry=80x10 -e 'sh -c  "echo STARTING Main Script; ruby TitleGrabber.rb -l '$ListSplit' >> '$Script_LogsFile'"'
        fi
        if [ $ToSplit == no ]; then
            gnome-terminal --geometry=80x10 -e 'sh -c  "echo STARTING Main Script; ruby TitleGrabber.rb -l '$List' >> '$Script_LogsFile'"'
        fi
    fi
    
    NetworkOutage=$(ifconfig ${NetworkAdapter} | grep "inet addr" | wc -l)
    if [ $NetworkOutage == 0 ]; then
        echo "Network Disconnected!"
        echo "Network Disconnected: EXIT: Logger, Script.....................$(date)"
        echo "Network Disconnected: EXIT: Logger, Script.....................$(date)" >> $LogsFile
        pkill less
        pkill ruby
        exit
    fi
fi
done
```

### TitleParser:

I extended the original project a little bit and added a TitleParser section.

Alrighty, now that we have a Database.csv file from TitleGrabber. We'll need to parse out the data into sql statements with creds.
I'm still currently working on a one-script does it all, but for now this is how it's done. With the creds.csv - parse.sh - Database.csv - Loop.sh file all in the same directory.

You'll be running: '**bash Loop.sh**'

Essentially, Loop.sh is running a series of "'**time parallel bash parse.sh Database.csv <outputfile> ::: {lines}**'"
This will parse the file in bulk of 25k at a time.

To make sure the files are good: '**grep -in "^[^(]" *.sql**'
^Checks the file to make sure it starts with '(', if it doesn't that sql file is not good and will need to be fixed, grep will post the line number to fix it. (The long sed command below fixes all of this)

You will also want to run: '**grep -in "(''," *.sql**'
^This checks for any lines without an ID set. (The long sed command below fixes all of this)

Loop.sh will output a series of .sql files, with 25k sql statements in each.

Once you have all the sqlfiles, use the command below that will add the first sql statement in, and change the last character of the file.

 sed -i '1iINSERT INTO `database` (`id`, `linenum`, `dates`, `ip`, `hostname`, `tcp_port`, `service`, `version`, `extra_info`, `product`, `state`, `ip_url`, `title`, `creds`) VALUES' *.sql && sed -i '$s/,$/;/' *.sql && sed -i "s/, ,/,0,/g" *.sql && sed -i "s/('',/('0',/g" *.sql && sed -i "s/('/('0','/g" *.sql

## Setting up MySQL Database:

You'll need to make a database for TitleGrabber, you can use the statement below to get the database up.

```sql
 --
 -- Database for TitleGrabber
 --
 create database TitleGrabber;
 
 DROP TABLE IF EXISTS `database`;
 
 use TitleGrabber;
 
 CREATE TABLE `database` (
 	`id`         int(10) NOT NULL auto_increment,
 	`linenum` varchar(250) NOT NULL default '',
 	`ip` varchar(250) NOT NULL default '',
 	`hostname`  varchar(250) NOT NULL default '',
 	`tcp_port`   int(8) DEFAULT NULL,
 	`service`      varchar(250) NOT NULL default '',
 	`version`     varchar(250) NOT NULL default '',
 	`extra_info`        varchar(250) NOT NULL default '',
 	`product`     varchar(250) NOT NULL default '',
 	`state`        varchar(250) NOT NULL default '',
 	`ip_url`        varchar(250) NOT NULL default '',
 	`title`        mediumtext,
 	`dates` datetime default NULL,
 	`creds` mediumtext,
 	PRIMARY KEY  (`id`),
 	UNIQUE KEY  (`id`)
 );
 ```

DO NOTE: You might need to up the MYSQL limit to import the files, in that case use:

**SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));**

Afterwards, you can now use the linux command: '**mysql -u root -p TitleGrabber < file.sql**'
^Might take awhile since you'll need to go through ~17 commands for each 25k file.

Instead you can log into mysql: '**mysql -uroot -p --database=TitleGrabber**'
and than use: '**source file1.sql; source file2.sql, source file3.sql;**'

Doing the source command manually will take awhile, you can use the '**Sql_Inject.sh**' file to generate the full command. Note: All the sql files should be in it's own directory, in my case I put them all into SQL. Put the script in that directory and run it.

DO NOTE: Sql_Inject.sh also has a function that will let you know if you need to manually fix any lines, this fixes are more single quotes perline than what sql needs. This is rare and will most likely happen to 1 or two lines total.

The script is located at my github: [https://github.com/Techryptic/Parallel-Execution-of-Selenium-Webdriver-Using-Multithreading](https://github.com/Techryptic/Parallel-Execution-of-Selenium-Webdriver-Using-Multithreading)
