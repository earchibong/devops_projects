## SHELL SCRIPTING - ONBOARD NEW LINUX USERS TO SERVER

## Step One: Create CSV File
- make a directory named `shell` create a file `names.csv`:
  - `mkdir shell`
  - `cd shell`
  - `touch names.csv`
- Add some names (one per line) into `names.csv`

![names](https://user-images.githubusercontent.com/92983658/179495557-c3a98657-77a9-4c67-ac76-97c34b2cb334.png)

- create a new group `developers` in `shell` directory : 
  - `sudo groupadd developers`

## Step Two: Create Bash Script
- create a new file in `shell` directory: `touch onboarding_users.sh`
- specify shell path:  `which bash`
- open `onboarding users` : `vim onboarding users`
- add shebang and shell path at top: `#!/usr/bin/bash`
- create variable `filein` and store `names.csv` in variable: `filein="names.csv"`
- create an input field separator using a new line: `IFS=$'\n'`
- create an if statement to check if the file we're trying to run exists:
  - if it doesn't exist, print out : `cannot find file`
  - if it does exist...
   - create an array of names and groups
   - create groups from the groups array
    - check if the group exists, if not create it
   
 
```

if [ ! -f "$filein" ]
then
  echo "Cannot find file $filein"
else

  #create arrays of groups, names
  groups=(`cut -d: -f 2 "$filein" | sed 's/ //'`)
  names=(`cut -d: -f 1 "$filein"`)

  #checks if the group exists, if not then creates it
  for group in ${groups[*]}
  do
    grep -q "^$group" /etc/group ; let x=$?
    if [ $x -eq 1 ]
    then
      groupadd "$group"
    fi
  done
  
  ```
  
  ![groups](https://user-images.githubusercontent.com/92983658/179508957-33f89ba3-3df4-4eb5-bc9a-313feedfd337.png)

  
  - then create user accounts and add them to groups in the same `onboarding_users` file
  
  ```
  
  x=0
  created=0
  for name in ${names[*]}
  do
    useradd -n -c ${names[$x]} -g "${groups[$x]}" $name 2> /dev/null
    if [ $? -eq 0 ]
    then
      let created=$created+1
    fi
#  echo "Name: ${names[$x]},  pw: ${names[$x]}"
    #This creates the password for the user suppresses output of passwd
    #The -p in the useradd function doesn't set it properly
    echo "${names[$x]}" | passwd --stdin "$name" > /dev/null
fi

```

![users](https://user-images.githubusercontent.com/92983658/179509003-d63e236c-3b4f-4160-a220-c7bec346a658.png)


