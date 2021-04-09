# INTRODUCTION
GFS shares many of the same goals as previous distributed file systems such as performance, scalability, reliability, and availability.
1. First, component failures are the norm rather than the exception. Therefore, constant monitoring, error detection, fault tolerance, and automatic recovery must be integral to the system.
2. Files are huge by traditional standards. Multi-GB files are common.
3. Third, most files are mutated by appending new data rather than overwriting existing data.
4. Fourth, co-designing the applications and the file system API benefits the overall system by increasing our flexibility.

# DESIGN OVERVIE
## Aussumptions
1. The system is built from many inexpensive commodity components that often fail.
2. The system stores a modest number of large files.
3. The workloads primarily consist of two kinds of reads: large streaming reads and small random reads.
4. The workloads also have many large, sequential writes that append data to files.
5. The system must efficiently implement well-defined semantics for multiple clients that concurrently append to the same file.
6. High sustained bandwidth is more important than low latency.

## Architecture
Files are divided into fixed-size chunks. Each chunk is identified by an immutable and globally unique 64 bit chunk handle assigned by the master at the time of chunk creation. For reliability, each chunk is replicated on multiple chunkservers. By default, we store three replicas.

The master maintains all file system metadata. This in- cludes the namespace, access control information, the map- ping from files to chunks, and the current locations of chunks.

Clients interact with the master for metadata opera- tions, but all data-bearing communication goes directly to the chunkservers.

Neither the client nor the chunkserver caches file data.


