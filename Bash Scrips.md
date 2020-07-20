# Bash Scrips
#pentest/tryhackme

## Find gtfobins
```bash

$ find / -perm /4000 2>/dev/null | xargs -I {} basename {} | xargs -I {} rg {} gtfobins.txt
```

## Making it easier to use LinkFinder
```sh
#!/bin/bash

GREEN='\033[0;32m'
YELLOW='\033[0;33m'
CYAN='\033[0;36m'
END='\033[0m'

if [[ $# -eq 0 ]] ; then
    printf '\nNo Host File or URLs file Given!'
    printf '\n\nUsage: jsrecon <path-to-urls-file>\n\n'
    exit 0
fi

printf "${YELLOW}[+]${END} jsrecon started.\n"

mkdir -p jsrecon_results
mkdir -p jsrecon_results/js
mkdir -p jsrecon_results/db

linkf=~/LinkFinder/linkfinder.py

for i in $(cat $1)
do
        cd jsrecon_results
        n1=$(echo $i | awk -F/ '{print $3}')
        n2=$(echo $i | awk -F/ '{print $1}' | sed 's/.$//')
        mkdir -p js/$n1-$n2
        mkdir -p db/$n1-$n2
        timeout 60 python3 $linkf -d -i $i -o cli > js/$n1-$n2/raw.txt

        jslinks=$(cat js/$n1-$n2/raw.txt | grep -oaEi "https?://[^\"\\'> ]+" | grep '\.js' | grep "$n1" | sort -u)

        if [[ ! -z $jslinks ]]
        then
                for js in $jslinks
                do
                        python3 $linkf -i $js -o cli >> js/$n1-$n2/linkfinder.txt
                        echo "$js" >> js/$n1-$n2/jslinks.txt
                        wget $js -P db/$n1-$n2/ -q
                done
        fi
        cd ..
        printf "${GREEN}[+]${END} $i ${YELLOW}done${END}.\n"
done

printf "${YELLOW}[+]${END} Script is done.\n"
printf "\n${YELLOW}[+]${END} Results stored in jsrecon_results.\n"
```