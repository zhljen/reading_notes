##Time and speek
	1ns = 10^-9s, 1us = 10^-6s, 1ms = 10^-3s
   |Item               | Time             |
   |processor cycle    | 0.5ns (2GHz)     |
   |cache access       | 1ns (1GHz)       |
   |memory access      | 15ns             |
   |context switch     | 5000ns ( 5us )   |
   |disk access        | 7000000ns (7ms)  |
##programming image in memory
high address -- command-line arguments and env variables
                stack  
                  |
                 ....
                heap  ( allocations from malloc family )
                uninitialized static data
                initialized static data
low address  -- program text 

##UNIX file implementation
inode (128 bytes): file information ( size, owner, link, permissions 68 bytes ), direct pointers(12 * 4 byte ), single indirect pointer (4byte), double indirect pointer(4byte), triple indirect pointer(4byte)
block 8K, then (1) 8kX12= 98304 is the file size represented by the direct pointers.
               8192/4 = 2048 (2k) pointers   
               (2) 2k x 8k = 16M 
               (3) 2k x 2k x 8k = 32G 
               (4) 2k x 2k x 2k x 8k = 64T 
