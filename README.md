Sparse matrix SpMV Project (PARCO-D1)
[made by Lorenzo Vadacca]

 ---
(All the files are zipped for dimension problems. Please download.)

## 1\. Compiler Version and Flags

This project was developed and tested using:

Compiler: GCC 7.5.0

Parallelization: OpenMP support enabled with `-fopenmp`

Flags used:

-O3 -fopenmp -std=c++98

⦁	"-O3" → Maximum optimization
⦁	"-fopenmp" → OpenMP support
⦁	"-std=c++98" → C++ standard

---

## 2\. How to Compile and Run

 Local

2 possible ways:

1. Run with .bat file:

   Double click on the "run_spmv.bat"

   or

2. Run from terminal: 

   "cd PathOfProject

   run_spmv.bat"

   (To modify parameters open the "run_spmv.bat" with Block Notes or equivalent)

 - Execute the file from terminal:

   "g++ -O3 -fopenmp -std=c++98 src/main.cpp src/csr.cpp src/spmv.cpp -o spmv.exe"

    Pass from terminal the parameters you want to modify

   "./spmv.exe matrices/large_matrix.mtx --threads 16 --runs 10 --schedule dynamic --chunk 1000 --output results.csv"

(! It is mandatory for all local methods to modify the Scheduling type and the Chunk size from the source code, from the "main.cpp" file and the "spmv.h" file.)

 - Cluster

1. Submit PBS job:

   "dos2unix run_cluster.pbs"

   "qsub run_cluster.pbs"
 
   The PBS script handles compilation, module loading, and looping through matrix files, thread counts, scheduling types, and chunk sizes.

   (You can modify the parameters from inside the "run_cluster.pbs" file with Block Notes or equivalent)

---

## 3\. Input and Output

Input

 Matrix file in Matrix Market (.mtx) format

 __________________________________________________________________________
| Matrix file 		         | Matrix size (rows , columns, non-zero elements) |
|------------------------|-------------------------------------------------|
|small_matrix.mtx 	      | 5, 5, 9	       	                 
|small_medium_matrix.mtx | 500, 500, 6700	                   
|medium_matrix.mtx 	     | 10000, 10000, 200000              
|medium_large_matrix.mtx | 50000, 50000, 1580000             
|large_matrix.mtx 	      | 100000, 100000, 2000000     	     
 

---

Runtime parameters:

⦁"--threads" → number of OpenMP threads
⦁"--runs" → number of benchmark runs
⦁"--schedule" → scheduling type (`static`, `dynamic`, `guided`)
⦁"--chunk" → chunk size for OpenMP
⦁"--output" → CSV filename

Output

⦁CSV format with columns:

  "Matrix,Threads,Schedule,Chunk,Runs,Avg(ms),Min(ms),Max(ms),90th(ms),Speedup"

---

## 4\. Modifying Parameters

⦁	Matrix sizes: change the `MATRICES` variable in the PBS script or provide a different ".mtx" file
⦁	Number of threads / CPUs:** adjust `THREADS` variable in PBS script or use `--threads` in CLI
⦁	Chunk sizes & scheduling:** modify `CHUNK_LIST` and `SCHEDULINGS` arrays in PBS script or pass as CLI arguments

Default values in code:

⦁	Threads: 1
⦁	Runs: 10
⦁	Schedule: static
⦁	Chunk: 1

- Adviced Chunk set for every matrix file

______________________________________________________
| Matrices		            | Rows	  | Adviced chunk   		  \
|-----------------------|-------------------------------|
| small_matrix		        | 5	     | 1, 2, 5			              
| small_medium_matrix	  | 500	   | 1, 10, 50, 100			        
| medium_matrix		       | 10000	 | 10, 100, 1000			        
| medium_large_matrix	  | 50000	 | 100, 1000, 5000		        
| large_matrix		        | 100000 | 100, 1000, 5000, 10000    


---

## 5\. Implemented functionalities:

  _____________________________________________________________________________________________________________________
 / Step 			                    | What it does in the code 				      	                                   			            \ 
|------------------------------|----------------------------------------------------------------------------------------|
| 1. Read matrix `.mtx` 	      | `loadMatrixFromMTX(filename)` read file, ignore comments `%`, read dimensions and triple 
| 2. Conversion in CSR         | Sort of triple by row/column, construction of  `row_ptr`, `col_idx` e `vals` 	            
| 3. Generating casual vector  | `vector<double> x(A.cols)` with `rand()` e constant seed			 	                            
| 4. SpMV sequential 		        | `spmv_sequential(A, x)` calculate `y_seq` 						                                     
| 5. SpMV parallel (OpenMP) 	  | `spmv_parallel(A, x, num_threads)` calculate `y_par` 				                              
| 6. Time measurement 		       | `omp_get_wtime()` before and after every execution, loop for `runs` iteractions            
| 7. Time statistics     	     | Calculate average, min, max, 90° percentile  					                                    
| 8. Verify correctness 	      | Calculate `max_diff` between `y_seq` and `y_par` 					                           
| 9. Output first value 	      | Print first 10 elements of `y_par`    						                                          
| 10. Save results CSV         | Write of `results.csv` with all parameters and statistics


---

## 6\.  Cluster-specific Notes

⦁	Modules to load: "module load gcc75"
⦁	Queue: "short_cpuQ"
⦁	Resources: "#PBS -l select=1:ncpus=16:mem=4gb"
⦁	Walltime: "#PBS -l walltime=02:00:00"
⦁	Output directory: make sure "$PBS_O_WORKDIR" is writable
⦁	File names: ensure unique CSV/output file names to prevent overwriting previous results
⦁ 	The PBS script loops through all scheduling/threads/chunks automatically.
