Sparse Matrix–Vector Multiplication with MPI (PARCO – Deliverable 2)\
<ins>[made by Lorenzo Vadacca-256374]</ins>
> [!NOTE]
> All the files are zipped for dimension problems. Please download "Vadacca-256374-ParcoD2.7z".

This project implements a distributed Sparse Matrix–Vector multiplication (SpMV) using the Compressed Sparse Row (CSR) format and MPI for parallelization.
The work follows Foster’s methodology for parallel program design and includes strong and weak scaling experiments executed on an HPC cluster.

---
## 1. Compiler, MPI Library, and Flags

The project was developed and tested on an HPC cluster using:

- Compiler: GCC 9.1.0

- MPI implementation: MPICH 3.2.1

- Language standard: C++11

Compilation flags:

"O2 -std=c++11"

- O2 → compiler optimizations

- std=c++11 → C++ standard compliance

MPI compilation is handled via mpirun or equivalent MPI wrappers.

---
## 2. Project Structure

Deliverable2/ \
├── matrices/      # Matrix Market (.mtx) input files \
| 	   ├── strong/ \
|	    ├── weak/\
├── src/           # C++ source files \
│	    ├── main.cpp \
│	    ├── csr.cpp \
│	    ├── spmv.cpp \
│	    ├── partition.cpp \
│	    ├── csr.h \
│	    ├── spmv.h \
│	    └── partition.h \
├── scripts/       # PBS scripts for cluster execution \
│	    └── cluster.pbs \
└── README.md \

---
## 3. How to Compile and Run

All experiments are executed on the cluster using a PBS job script.

From the scripts/ directory:

- dos2unix cluster.pbs
- qsub cluster.pbs


The PBS script:

- 1)Loads compiler and MPI modules

- 2)Compiles the code

- 3)Executes strong scaling and weak scaling benchmarks

- 4)Collects timing results

All paths are resolved relative to the project root.

---
## 4. Input and Output

Input: Sparse matrices in Matrix Market (.mtx) format

Matrices used in the experiments:

|Matrix file		              |Rows  |	Cols |	NNZ  |
|---------------------------------|------|-------|-------|
|small_matrix.mtx	              |10k   | 10k   | 200k   
|small_medium_matrix.mtx          |14k   |	14k  | 400k  
|medium_matrix.mtx	              |20k   |	20k  | 800k  
|medium_large_matrix.mtx   	      |28k   |	28k  | 1.6M   
|2nd_small_matrix.mtx             |40k   |	40k  | 3.2M   
|2nd_small_medium_matrix.mtx      |57k   |	57k  | 6.4M   
|2nd_medium_matrix.mtx	          |80k   |	80k  | 12.8M  
|2nd_medium_large_matrix.mtx      |116k  |	116k | 20M    
|large_matrix.mtx	       		  |130k  |	130k | 25M   
|strong_medium_large_matrix.mtx   |50k   |	50k  | 1.58M 
|strong_large_matrix.mtx          |100k  |	100k | 25M   

Synthetic matrices were generated to support weak scaling up to 256 MPI processes.

Output

The program prints:

- Matrix dimensions and number of non-zeros
- Maximum execution time across ranks
- Scaling results
- Benchmark results are stored in CSV format, including:
- Matrix,Processes,Rows,Cols,NNZ,Time(s),Speedup,Efficiency

---
## 5. Implemented Functionality

|Step  |Description			    			 |
|------|------------------------------------ |
|1     |Rank 0 reads .mtx file		    	 |
|2     |Matrix converted from COO to CSR     |
|3     |1D row-wise partitioning	    	 |
|4     |Identification of ghost columns      |
|5     |MPI communication of vector elements |
|6     |Local SpMV on each rank		    	 |
|7     |Timing via MPI_Wtime()		    	 |
|8     |Strong and weak scaling benchmarks   |
|9     |Speedup and efficiency computation   |
|10    |CSV output generation		    	 |

---
## 6. Parallel Design (Foster’s Methodology)

The implementation follows Foster’s four steps:

1.	Partitioning:
Rows distributed across MPI ranks (1D decomposition).
2.	Communication:
Exchange of ghost elements of the input vector.
3.	Aggregation:
Local CSR matrices built per rank to reduce communication.
4.	Mapping:
    MPI ranks mapped directly to compute processes.

---
## 7. Scaling Experiments

⦁	Strong Scaling

Fixed matrix size

Number of MPI processes increased from 1 to 256

Goal: measure reduction in execution time

⦁	Weak Scaling

Matrix size increases proportionally to the number of processes

Constant workload per rank

Goal: assess parallel efficiency at scale

---
## 8. Performance Metrics

The following metrics are reported:

|Type					| Formula          |
|-----------------------|------------------|
|Speedup:		  		| 𝑆(𝑃) = 𝑇(1)/𝑇(𝑃) |​
|Efficiency: 	  		|	𝐸(𝑃)=𝑆(𝑃)/𝑃	   |	​
|Estimated FLOPs:  		| 2×𝑁𝑁𝑍		   |
|Execution time per SpMV|				   |

---
## 9. Cluster Notes

Modules required:

"
module load gcc91
module load mpich-3.2.1--gcc-9.1.0
"

⦁	Queue: short_cpuQ

⦁	Walltime: configurable in PBS script

⦁	Max tested scale: 256 MPI processes

---
## 10. Notes

Synthetic matrices are used to reach large process counts.

Results are reproducible due to fixed random seeds.

The implementation prioritizes clarity and correctness over aggressive low-level optimizations.
