# Week 1

## Lesson 1.1: Spark

### Motivation

- Iterative algorithms and interactive data exploration are commonly used in many domains
- ***Traditional MapReduce and classical parallel runtimes cannot solve iterative algorithms efficiently***...why?
  - Hadoop : Repeated data access to HDFS, no optimization to data caching and data transfers
  - MPI : No natural support of fault tolerance and programming interface is complicated

### Retrofitting Iterating on MR

- MR does not support iteration out of the box
- But we still want Page rank, clustering etc.
  - Mahout
  - Nutch
- The "obvious" solution: Split iteration into multiple mapReduce jobs. Write a driver program for orchestration.