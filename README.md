# CSC453 - Program 3: Memory Simulator

Names: Rianna Lei & Nathan Luong

## COMPLETED:
- [x] Address bit-masking and decoding
- [x] TLB and Page Table logic
- [x] Physical memory and Backing Store management
- [x] FIFO page replacement algorithm and variable frame support
- [x] Output formatting and summary statistics
- [x] LRU page replacement algorithm
- [x] OPT page replacement algorithm

## NOTES:
- We implemented the TLB as a FIFO cache with 16 entries as specified
- The Page Table includes a "present" bit to track residency.
- Hits in the FIFO algorithm do not change the position in the queue.
- TLB is flushed when a page is evicted from physical memory to maintain consistency.

## RUN THE PROGRAM:
Make the script executable and run it using the following format:

```bash
chmod +x memSim.py
./memSim.py <reference-file.txt> <FRAMES> <PRA>
