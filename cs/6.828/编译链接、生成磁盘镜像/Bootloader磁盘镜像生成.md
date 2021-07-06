```C
echo -n -e "\x55\xaa" | dd ibs=2 obs=1 of=./51 count=1 seek=510 conv=notrunc
```

