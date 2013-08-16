---
categories: blog
title: Area Resource File for R
author: Bjorn Arneson
layout: post
tags: 
- R
- statistics
- ARF
- area resource file
- script
- conversion
---

I've been mucking around in R recently trying to teach myself a few new tricks related
to [my job](http://www.institute.optum.com). In the course of Googling around, I found that
[Biostat Matt](http://biostatmatt.com/archives/932) put together a slick
script for converting the 2009 Area Resource File (ARF) from a SAS file into a format that R
can consume.

> The file is difficult to work with directly because of its size and format. Because the data are stored as text, numbers are stored inefficiently (after conversion and compression, the equivalent R data file is 10% of the original size). In cases like this, the saving-grace is human readability. Although the file is ASCII, or rather extended ASCII (I found an accent in San Sebastiàn, PR), it's not human-readable because the 6256 fields aren't delimited and are variable in length. Hence, it's nearly impossible to visually track where fields begin and end. The data are distributed with a SAS macro to read the data into a SAS dataset.

The ARF is a ridiculously comprehensive database of health care statistics for every
county in the U.S. It is a huge dataset--the 2011-12 version comprises 3231 observations 
of 6261 variables.

I updated Matt's C script for the [2011-2012 ARF](http://datawarehouse.hrsa.gov/datadownload/ARF/arf2011-2012.zip)
--the first part of my update is shown
below. In order to use the script, you'll have to [download the full file from this 
gist](https://gist.github.com/2914494). (I left out some important bits for the sake
of readability on this page.)

{% highlight c %}
/* gcc -o asc2csv asc2csv.c */
/* ./asc2csv arf2011.asc > arf2011.csv */

#include <stdio.h>
#include <stdlib.h>

#define NCOL 6261
#define RLEN 31897
static unsigned short len[NCOL];
static char * lab[NCOL];
int main(int argc, char** argv) {
    FILE * fd;
    char buf[RLEN+2], *ptr;
    unsigned int num, col;
    if(argc <2) {
        printf("no file specified\n");
        exit(1);
    }

    fd = fopen(argv[1], "rb");
    if(!fd) {
        printf("file open failed\n");
        exit(1);
    }

    for(col = 0; col < NCOL; col++) {
        if(col < NCOL - 1) {
            printf("\"%s\",", lab[col]);
        } else {
            printf("\"%s\"\n", lab[col]);
        }
    }

    while(!feof(fd)) {
        if(fread(buf, 1L, RLEN+2, fd) < RLEN+2) {
            printf("file read failed\n");
            exit(1);
        }
        for(ptr = buf, col = 0; col < NCOL; col++) {
            printf("\"");
            fwrite(ptr, 1L, len[col], stdout);
            ptr += len[col];
            if(col < NCOL - 1) {
                printf("\",");
            } else {
                printf("\"\n");
            }
        } 
    }
    return 0;
}
</code>
{% endhighlight %}