#!/usr/bin/env python3
import sys
from collections import deque

# MEMORY STRUCTURES
class TLB:
    """
    Translation Lookaside Buffer: A small cache for page-to-frame mappings.
    Spec: 16 entries, FIFO replacement.
    """
    def __init__(self):
        self.size = 16
        self.entries = deque() # Stores [page_num, frame_num]
        self.hits = 0
        self.misses = 0

    def lookup(self, page_num):
        """ Returns frame_num on hit, else None. """
        for entry in self.entries:
            if entry[0] == page_num:
                self.hits += 1
                return entry[1]
        self.misses += 1
        return None

    def update(self, page_num, frame_num):
        """ FIFO Update: If full, pop the oldest. Hit does NOT change order. """
        # Check if already in TLB (spec says don't change position on hit)
        for entry in self.entries:
            if entry[0] == page_num:
                return
        
        if len(self.entries) >= self.size:
            self.entries.popleft() # Evict oldest entry
        self.entries.append([page_num, frame_num])

    def remove(self, page_num):
        """ TLB Consistency: Remove a page if it gets evicted from RAM. """
        for entry in list(self.entries):
            if entry[0] == page_num:
                self.entries.remove(entry)
                break

class PageTable:
    """
    Page Table: Keeps track of all 256 pages and their residency
    """
    def __init__(self):
        self.table = [None] * 256
        self.present = [False] * 256
        self.faults = 0

    def lookup(self, page_num):
        """ Returns frame_num if resident, else registers a fault """
        if self.present[page_num]:
            return self.table[page_num]
        self.faults += 1
        return None

    def update(self, page_num, frame_num):
        self.table[page_num] = frame_num
        self.present[page_num] = True

def opt_select_victim(frame_to_page, current_index, page_sequence, frame_load_order):
    furthest = -1
    victim_frame = 0

    for frame, page in enumerate(frame_to_page):
        if page is None:
            return frame
        
        next_use = float('inf')
        for j in range(current_index + 1, len(page_sequence)):
            if page_sequence[j] == page:
                next_use = j
                break

        if next_use > furthest or (next_use == furthest and frame_load_order[frame] < frame_load_order[victim_frame]):
            furthest = next_use
            victim_frame = frame
    
    return victim_frame

# SIMULATOR ENGINE
def main():
    # Handle Command Line Args
    if len(sys.argv) < 2:
        print("Usage: ./memSim <reference-file.txt> <FRAMES> <PRA>")
        return

    ref_file = sys.argv[1]
    # Defaults: 256 frames, FIFO algorithm
    num_frames = int(sys.argv[2]) if len(sys.argv) > 2 else 256
    algorithm = sys.argv[3].upper() if len(sys.argv) > 3 else "FIFO"

    # Initialize Hardware Components
    tlb = TLB()
    page_table = PageTable()
    phys_mem = [bytearray(256) for _ in range(num_frames)]
    
    # Trackers for Replacement Logic
    frame_to_page = [None] * num_frames
    fifo_pointer = 0 # Points to the next frame to be replaced
    frame_load_order = [0] * num_frames
    load_counter = 0
    frames_filled = 0
    lru_timer = 0
    frame_last_used = [0] * num_frames

    # Load Addresses
    try:
        with open(ref_file, 'r') as f:
            addresses = [int(line.strip()) for line in f if line.strip()]
    except FileNotFoundError:
        print(f"Error: {ref_file} not found.")
        return
    
    page_sequence = [(addr >> 8) & 0xFF for addr in addresses]

    # Start Simulation Loop
    for current_index, addr in enumerate(addresses):
        # Address Decoding: 16-bit address (8-bit page, 8-bit offset)
        page_num = (addr >> 8) & 0xFF
        offset = addr & 0xFF
        
        # 1. Search Order: Check TLB
        frame_num = tlb.lookup(page_num)
        
        if frame_num is None:
            # 2. TLB Miss: Check Page Table
            frame_num = page_table.lookup(page_num)
            
            if frame_num is None:
                # 3. PAGE FAULT: Pull from Backing Store
                with open("BACKING_STORE.bin", "rb") as bin_file:
                    bin_file.seek(page_num * 256)
                    page_data = bin_file.read(256)
            
                if algorithm == "FIFO":
                    frame_num = fifo_pointer
                    fifo_pointer = (fifo_pointer + 1) % num_frames
                elif algorithm == "OPT":
                    frame_num = opt_select_victim(frame_to_page, current_index, page_sequence, frame_load_order)
                
                # Check if we have empty frames before starting eviction logic
                if frames_filled < num_frames:
                    frame_num = frames_filled
                    frames_filled += 1
                else:
                    # Use Replacement Algorithm to choose frame
                    if algorithm == "FIFO":
                        frame_num = fifo_pointer
                        fifo_pointer = (fifo_pointer + 1) % num_frames
                    elif algorithm == "LRU":
                        frame_num = frame_last_used.index(min(frame_last_used))
                    elif algorithm == "OPT":
                        frame_num = opt_select_victim(frame_to_page, current_index, page_sequence, frame_load_order)

                
                # EVICTION: Clean up the old page mapping
                old_page = frame_to_page[frame_num]
                if old_page is not None:
                    page_table.present[old_page] = False
                    tlb.remove(old_page) # Consistency Tweak: Flush TLB!
                
                # load new data into Physical Memory
                phys_mem[frame_num] = bytearray(page_data)
                frame_to_page[frame_num] = page_num
                load_counter += 1
                frame_load_order[frame_num] = load_counter
                page_table.update(page_num, frame_num)
            
            # Update TLB after Page Table hit or fault
            tlb.update(page_num, frame_num)

        # IMPORTANT: LRU must update the timestamp on EVERY access (Hit or Miss)
        frame_last_used[frame_num] = lru_timer
        lru_timer += 1

        # retrieve referenced byte as a SIGNED integer (-128 to 127)
        raw_val = phys_mem[frame_num][offset]
        signed_val = raw_val if raw_val < 128 else raw_val - 256
        
        # format output: uppercase hex with no spaces
        hex_content = phys_mem[frame_num].hex().upper()
        print(f"{addr}, {signed_val}, {frame_num}, {hex_content}")

    # SUMMARY STATS
    total_addr = len(addresses)
    print(f"Number of Translated Addresses = {total_addr}")
    print(f"Page Faults = {page_table.faults}")
    print(f"Page Fault Rate = {page_table.faults / total_addr:.3f}")
    print(f"TLB Hits = {tlb.hits}")
    print(f"TLB Misses = {tlb.misses}")
    print(f"TLB Hit Rate = {tlb.hits / total_addr:.3f}")

if __name__ == "__main__":
    main()