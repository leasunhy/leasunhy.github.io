---
title: Find Prime Numbers in Parallel Using MPI and C++
date: 2015-10-21 22:50:28
tags:
  - MPI
  - C++
---

For the time being, I don't have enough time to explain the idea behind the program.

Hope the source file be self-explaining enough :-)

<!-- more -->

```c++
/**
 *
 * @file        : prime.cpp
 * @version     : 1.0
 * @description : Find out all primes smaller than MAXN in parallel using MPI and C++.
 * @author      : Leasunhy <leasunhy@gmail.com>
 * @date        : 2015-10-21
 * @license     : MIT License
 *
 * @section usage:
 *      First compile the source file:
 *          $ mpicxx -g prime.cpp -o prime
 *      Then run it with `mpirun`:
 *          $ mpirun -np 10 ./prime 1000000
 *      Then the program will run and find primes under 1000000 using 10 processes.
 *
 */

#include <cmath>
#include <sstream>
#include <iostream>
#include <algorithm>
#include <vector>
#include <iomanip>
#include <mpi.h>

using std::cout;
using std::cerr;
using std::endl;
using std::vector;

// MAXN: The upper bound of primes to be found.
int MAXN = 10000;
const int SIGNAL_PRIME_ALL_SENT = -1;

void mainProc(const MPI::Intracomm& comm);
void workerProc(const MPI::Intracomm& comm, int rank);

/**
 * main(argc, argv)
 *   checks the arguments passed from the command line and dispatch tasks.
 */
int main(int argc, char * argv[]) {
    MPI::Init(argc, argv);

    MPI::Intracomm comm = MPI::COMM_WORLD;
    if (comm.Get_size() == 1) {
        cerr << "Error: At least 2 processes are needed" << endl;
        MPI::Finalize();
        return 0;
    }
    int rank = comm.Get_rank();

    if (argc == 2) {
        std::stringstream ss(argv[1]);
        if (!(ss >> MAXN)) {
            if (rank == 0)
                cerr << "Usage: " << argv[0] << " MAXN" << endl
                     << "\tMAXN: the upper limit of numbers." << endl;
            MPI::Finalize();
            return 0;
        }
        if (MAXN < comm.Get_size()) {
            if (rank == 0)
                cerr << "Error: number of processes larger than MAXN!" << endl;
            MPI::Finalize();
            return 0;
        }
    }


    if (rank == 0)
        mainProc(comm);
    else
        workerProc(comm, rank);

    MPI::Finalize();
    return 0;
}

/**
 * mainProc(comm)
 *   The main process is responsible for distributing primes under sqrt(n).
 *   @param comm: the world Intracomm.
 */
void mainProc(const MPI::Intracomm& comm) {
    int end = std::ceil(std::sqrt(MAXN));
    vector<bool> portion(end + 1, true);
    portion[0] = portion[1] = false;
    for (int p = 2; p <= end; ++p) if (portion[p]) {
        comm.Bcast(&p, 1, MPI::INT, 0);
        for (int i = p + p; i <= end; i += p)
            portion[i] = false;
    }
    int signal = SIGNAL_PRIME_ALL_SENT;
    comm.Bcast(&signal, 1, MPI::INT, 0);

    vector<int> result;
    result.reserve(ceil(sqrt(MAXN)));
    for (int i = 1; i < comm.Get_size(); ++i) {
        int size;
        comm.Recv(&size, 1, MPI::INT, i, i);
        int ori_size = result.size();
        result.resize(result.size() + size);
        comm.Recv(result.data() + ori_size, size, MPI::INT, i, i);
    }
    // improve the speed of output
    std::ios::sync_with_stdio(false);
    for (size_t i = 1; i <= result.size(); ++i) {
        cout << std::setw(12) << result[i - 1];
        if (i % 10 == 0)
            cout << endl;
    }
    cout << endl;
}

/**
 * workerProc(comm, rank)
 *   The worker processes are responsible for filtering out the numbers divisible
 *      by the primes sent by the main process.
 *   @param comm: the world Intracomm.
 *   @param rank: the rank of the worker process.
 */
void workerProc(const MPI::Intracomm& comm, int rank) {
    int comm_size = comm.Get_size();
    int nums_per_rank = MAXN / (comm_size - 1);
    int begin = nums_per_rank * (rank - 1);
    int end = (rank == comm_size - 1) ? MAXN + 1 : begin + nums_per_rank;
    cout << "Rank " << rank << ": from " << begin << " to " << end - 1 << endl;
    vector<bool> portion(end - begin, true);
    if (rank == 1)
        portion[0] = portion[1] = false;

    while (true) {
        int p;
        comm.Bcast(&p, 1, MPI::INT, 0);
        if (p == SIGNAL_PRIME_ALL_SENT) break;
        for (int b = p + p; b < end; b += p) if (b >= begin)
            portion[b - begin] = false;
    }

    vector<int> res;
    for (int i = begin; i < end; ++i) if (portion[i - begin]) {
        res.push_back(i);
    }
    int count = res.size();
    comm.Send(&count, 1, MPI::INT, 0, rank);
    comm.Send(res.data(), count, MPI::INT, 0, rank);
}


```

