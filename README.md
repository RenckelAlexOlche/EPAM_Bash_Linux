# Task A (scan tcp/ip)

## Create a script that uses the following keys:
### 1. When starting without parameters, it will display a list of possible keys and their  description
### 2. The --all key displays the IP addresses and symbolic names of all hosts in the current subnet 
### 3. The --target key displays a list of open system TCP ports.The code that performs the functionality of each of the subtasks must be placed in a separate function+

```bash
#!/bin/bash

ping_curr_subN() {
	nmap -n -sP 192.168.1.0/24 > ip_logs
}

scan_ports() {
	nmap -p- 192.168.1.1 > ports_logs
	nmap -p- 127.0.0.1 >> ports_logs  
}

default=true

for arg; do
	case "$arg" in
		--all)
			ping_curr_subN 
			default=false
			;;
		--target) 
			scan_ports
			default=false
			;;
		*) echo "ERROR invalid input" ;;
	esac
done


if $default ; then
	echo "--all     do something"
	echo "--target  do another thing"	
fi
```


# Task B (scan logs)
 
## Using Apache log example create a script to answer the following questions:
### 1. From which ip were the most requests?
### 2. What is the most requested page?
### 3. How many requests were there from each ip?
### 4. What non-existent pages were clients referred to? 
### 5. What time did site get the most requests? 
### 6. What search bots have accessed the site? (UA + IP)

```bash
#!/bin/bash

#1 
awk '{ print $0  }' example_logs | grep -oP '((25[0-5]|(2[0-4]|1\d|[1-9]|)\d)\.?\b){4}' | sort -n | uniq -c | sort -nr | head -1 > result_logs
awk '{ new_var="1) the most requested ip: "$2; print new_var }' result_logs > ip_counter


#2
awk '{ print $0  }' example_logs | grep -oP ' \/[^\.]*\.html' | sort | uniq -c | sort -nr | head -1 > result_logs
awk '{ new_var="2) the most requested page: "$2; print new_var }' result_logs >> ip_counter


#3
echo "3)" >> ip_counter
printf "Count   IP\n" >> ip_counter
awk '{ print $0  }' example_logs | grep -oP '((25[0-5]|(2[0-4]|1\d|[1-9]|)\d)\.?\b){4}' | sort -n | uniq -c | sort -nr >> ip_counter
sed -n "/ 404 /p" example_logs > result_logs


#4
echo "4)" >> ip_counter
awk '{ print $0  }' example_logs | grep -oP '(GET|POST|PUT|DELETE) .* 404' >> ip_counter


#5
awk '{ print $0  }' example_logs | grep -oP '\[[^\[]*\+[^\[]*\]' > result_logs 
sed -ie 's/:/ /g' result_logs 
awk '{ print $2 }' result_logs | sort | uniq -c | sort -nr | head -1 > result_logs 
awk '{ new_var="5) the most requested time: "$2; print new_var  }' result_logs >> ip_counter


#6
sed -n '/ [^ ]*[Bb][Oo][Tt]\/.*;/p' example_logs > result_logs 
awk '{ print $0  }' example_logs | grep -oP '((25[0-5]|(2[0-4]|1\d|[1-9]|)\d)\.?\b){4}.* [^ ]*[Bb][Oo][Tt]\/.*;' > result_logs
echo "6)" >> ip_counter
awk '{ print $1" "$NF  }' result_logs | sort -n  | uniq | sort -nr >> ip_counter


# Remove the leftover files
rm result_logs
```


# Task C (create a data backup)

## Create a data backup script that takes the following data as parameters:
### 1. Path to the syncing directory.
### 2. The path to the directory where the copies of the files will be stored.
### 3. In case of adding new or deleting old files, the script must add a corresponding entry to the log file indicating the time, type of operation and file name. The command to run the script must be added to crontab with a run frequency of one minute

```bash
#!/bin/bash

#1
dir_path="$1"
echo $dir_path


#2
save_path="$2"
echo $save_path


#3 Create archive filename.
backup_date=$( date | tr " " _)
echo $backup_date
hostname=$(hostname -s)
archive_file="$hostname$backup_date.tgz"
echo $archive_file


# Backup the files using tar
find $dir_path -mmin -1 -print0 | xargs -0 tar fcz $save_path/$archive_file --absolute-names


# cron 
# * * * * * bash t_b_3 /home/user_name/path_of_data /home/user_name/dir_to_backup
```

