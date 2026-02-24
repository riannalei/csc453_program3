# CSC453 - Program 3: Memory Simulator

Names: Rianna Lei & Nathan Luong

COMPLETED:
- Address bit-masking and decoding
- TLB and Page Table logic
- Physical memory and Backing Store management
- FIFO page replacement algorithm and variable frame support
- Output formatting and summary statistics

TO BE COMPLETED:
- Implement LRU page replacement algorithm
- Implement OPT page replacement algorithm

RUN THE PROGRAM:
Make the script executable: chmod +x memSim.py
Run: ./memSim <addresses.txt> <FRAMES> <PRA>

NOTES:
- We implemented the TLB as a FIFO cache with 16 entries as specified
- The Page Table includes a "present" bit to track residency.
- Hits in the FIFO algorithm do not change the position in the queue.
- TLB is flushed when a page is evicted from physical memory to maintain consistency.