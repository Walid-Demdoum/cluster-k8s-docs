## Usefull commands for Linux
### SCP Command with port
```bash
scp <-p> <user@host:pathto> <user@target:pathto>
```
### Wget command to a login secure website
```bash
wget --user="email" --password="password" https://example.site
```

### Check differences between 2 repositoroes
```bash
diff -bur repository-1/ repository-2/
```

### Find file or directory with a directory path
```bash
find . -name "filename" /path/to/search
```

### Output difference of 2 files texts
```bashh
comm -23 file.txt file2.txt >> output.txt
```