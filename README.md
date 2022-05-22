# Extreme_Breach_Masks
A set of prioritized Hashcat masks intelligently developed from terabytes of password breach datasets and organized by run time.

## Goal
To improve the efficiency of password cracking using Hashcat mask attacks by prioritizing masks with the highest password cracking probability in the shortest possible using high volumes of password breach data.

## Background
Inspired by the work of golem445 who compiled a set of password hashcat password masks using real-world data. I took this a step further by building a set of prioritized Hashcat masks using an enormous password breach dataset that I have been personally compiling and curating.

## Methodology
1) Compiled every available password breach dataset that I could find -- terabytes of data!  Wordlists include everything readily google-able and torrent-able.  Noteable inclusions are: crackstation.net, seclists, rockyou, COMB, breach-parse... and many, many more.
2) Combined the wordlists in a way that they were generally sorted by password usage commonality.
3) Deduplicated the wordlist without re-sorting (important to retain the commonality order) using this tool: https://github.com/nil0x42/duplicut
4) Ran the wordlist through the statsgen.py tool to convert the wordlist into a counted set of password masks: https://github.com/iphelix/pack
```
python statsgen.py breach_wordlist.txt -o masks.statsgen
```
5) Ran the resulting statsgen.py output through maskgen.py to generate .hcmask files that are efficently ordered and seperated by run time. The run time duration assumes a hashing speed of 56,636,300,000 keys per second. This was determiend based on the performance of 1x Nvidia GTX1080Ti cracking NTLM hashes in Hashcat.  Example command below:
```
python maskgen.py --optindex -o ./1-hour_8.hcmask --minlength=8 --maxlength=8 --pps 56636300000 --targettime 3600 masks.statsgen
```
6) Repeated step #5 with various execution times to generate files optimized for various run times.

## Usage
The .hcmask files above describe passwords of differing character lenghts, each sorted by efficiency, and formatted for use by the Hashcat password cracking tool.  Depending on your situation, you might want to focus on passwords of a specific length only vs the entire set.  You should select the hcmask file optimized for your desired time frame.  The statssgen file is icnluded if you want to re-sort and generate your own hcmask files; however, I had to pair it down to only 8-14 characters and 7zip it because the full version was to large for github.  Recognize that this type of brute force mask attack can take a long time and should be performed last after you have exhausted more targeted methods.  My recommended password cracking attack order is below:

1) Basic dictionary attack with your favorite wordlist... ie rockyou.txt
```
hashcat.exe -d <include_gpu_numbers> -m 1000 -w 4 -a 3 --session <name_your_session> <ntlm_hashes.txt> <rockyou.txt> --force
```
2) Brute force all permutations 1-7 character length passwords... this does not take long given the minimal keyspace of this group.
```
hashcat.exe --increment --increment-min=1 -d <include_gpu_numbers> -m 1000 -w 4 -a 3 --session <name_your_session> <ntlm_hashes.txt> -1 ?l?u?d?s ?1?1?1?1?1?1?1 --force
```
3) Targeted dictionary attack... create a custom lowercase wordlist using CeWL and add local sports teams, city names, mascots, etc and apply the best64.rule
```
hashcat.exe -d <include_gpu_numbers> -m 1000 -w 4 -a 3 --session <name_your_session> <ntlm_hashes.txt> <custom_wordlist.txt> -r best64.rule --force
```
4) Analyze the set of cracked passwords for potential patterns, run targeted attacks which reflect those patterns.
5) BIG dictionary attack... run the passwords through the largest wordlist you have.
6) Analyze any newly cracked passwords for potential patterns, run targeted attacks which reflect those patterns.
7) Use this repository of work and run the "Efficient_#.hcmask" from this repo according to your needs.

## Example Hashcat Command for Using the .hcmask to Crack NTLM Hashes
```
hashcat.exe -d <include_gpu_numbers> -m 1000 -w 4 -a 3 --session <name_your_session> <ntlm_hashes.txt> 1-day_8-14.hcmask --force
```
