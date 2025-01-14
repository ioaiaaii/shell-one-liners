
# Shell One-liners and scripts

## Filesystem

### Files

Find files with specific type, containing a pattern
*e.g. get all `yml` files, containing networks `10.168` and `10.99`*

```shell
find . -type f -name "*.yml" -maxdepth 1 \
  -exec grep -q 10.168 {} \; -a \
  -exec grep -q 10.99 {} \; -a -print
```

Find all files here containing a specific string, and replace it.
*e.g. get all files recursively containing a port `6379` and replace it with `6399`*

```shell
find . -type f \                                                           
    -exec grep -r ' 6379' {} ';' \
    -exec sed -i '' 's/ 6379/ 6389/g' {} +
```

Iterate over an array and exec
*e.g. get the list of conf files, for every file print the filename and the content*

```shell
t=($(ls *.conf)) && for i in "${t[@]}";do echo $i && cat $i  ;done
```

Find number of files per dir
*e.g. find and print recursively the current number of files per dir*

```shell
for i in `ls -1A`; do echo "`find $i | sort -u | wc -l` $i"; done | sort -rn | head -5
```

Iterate over two files
*e.g for each line containing the hostname of `hostname_list_online`, find in the list `hostname_list_all` and print it.*

```shell
while read j; do [ -z "$(grep -w $j hostname_list_all)" ] && echo $j ; done < hostname_list_online
```

Iterate over multiple files, arrays and exec commands
*e.g. refresh `ex` and `tempfile` files, sort `tt` to `tempfile`, crete `tm` with a specific format`.*
*create an array with all lines , sort and clean it.*

*crete the new `ex` file, and remove each line of `ex` file, from `tempfile`.*

```shell
>  ex | > tempfile | sort -r -V tt > tempfile && \
tm=($(cat tempfile | awk -F ".v" '{print $1}')) && \
ttt=($(echo "${tm[@]}" | tr ' ' '\n' | sort -u | tr '\n' ' ')) && \
for i in "${ttt[@]}";do grep $i tempfile | head -n2 >> ex;done && \
while read j; do sed -i "/$j/d" tempfile; done < ex
```

### Performance

Get sum of RSS Memory

```shell
ps aux | awk '{x +=$6} END {print x/1024}'
```

## Kubernetes

Get annotation info for specific kind in all namespaces
*e.g. get value of ingress.class for all ingresses*

```shell
kubectl get ing -A -o json |  jq '.items[] | .metadata.annotations["kubernetes.io/ingress.class"]'
```

Actions against specific kind, based on condition.
*e.g. get pods with restarts > 10, delete them*

```shell
pods=($(kubectl get pods -A | awk '{if($5>10)print $1","$2}' | sed 1d)) && for i in "${pods[@]}";do ns=$(echo $i | awk -F "," '{print $1}') && pod=$(echo $i | awk -F "," '{print $2}') && echo "Deleting:  NAMESPACE $ns"  POD $pod && kubectl -n $ns delete pod $pod && sleep 3;done
```

## SSH

Run a local script with variables, as sudo on remote servers.

I. Create the script in a file `commands.sh`

e.g.

```shell
nc -v $1 22
```

II. Run the script against a list of servers.

```shell

USERNAME=<username>
MYCOMMAND=$(base64 commands.sh)

ips=($(get ips from somewhere| awk -F '|' '{print $5}' | grep -v "Private IP"))

for i in "${ips[@]}"
do
    ssh -i /path_to/.ssh/id_rsa -o StrictHostKeyChecking=no $USERNAME@$i " echo $MYCOMMAND | base64 -d | sudo bash "
done
```
## Processes
Get a pid and kill it
*e.g., get the PID of an SSH tunnel in the 8888 port and kill it*

```shell
ps aux | awk '/ssh/ && /8888:localhost:8888/ && !/awk/ { print $2 }' | xargs kill
```
