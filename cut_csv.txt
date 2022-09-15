#!/bin/bash
for filename in ~/Downloads/dailyradio/*.csv; do
	cut -d ',' -f1-35 "$filename" > output.csv
	mv output.csv "$filename"
done
