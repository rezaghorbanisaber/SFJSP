# SFJSP: benchmark instances, detailed results and statistical analyses

This repository contains the benchmark instances, the run-level results, the detailed results workbook, the figures and the statistical analyses of the computational experiments of

> R. Ghorbani Saber, P. Leyman, E.-H. Aghezzaf. *Stochastic Flexible Job Shop Scheduling Problem.* (Section 7)

The stochastic flexible job shop scheduling problem (SFJSP) assigns every operation to an eligible machine and sequences the operations on the machines before the processing times are known. The processing times are random, and every operation starts as soon as its job predecessor and its machine predecessor are finished. Three measures of the makespan distribution are optimized over a sample of scenarios (SAA problem): the expected makespan E[Cmax], the service level α(T) = P(Cmax ≤ T) for a deadline T, and CVaR_β(Cmax).

The source code is not included in this repository.

## Contents

| Path | Content |
|---|---|
| `Inputs/instances/` | the 87 benchmark instances of the experiments |
| `Inputs/instances.csv` | for every instance file: benchmark, original name, jobs, machines, operations, and the instance sets in which it is used |
| `Inputs/deadlines.csv` | nominal makespan Cmax(s0) of the deterministic solution s0 of every instance, which defines the deadline T |
| `Outputs/` | one CSV file per experiment with one row per run (see below) |
| `results/SFJSP_Section7_results.xlsx` | the detailed results workbook: all cases, every table and number of the text as worksheet formulas on the cases, the statistical tests and the data of all figures |
| `results/figures/` | the figures of Section 7 (PDF) |

## Experiments

| Experiment | Instances | Random operations | Scenarios \|Ω\| | Settings | Methods | Section |
|---|---|---|---|---|---|---|
| Small instances | Kacem (4), Fattahi (20), Hurink mt06, la01–la05 in edata, rdata, vdata (18) | all | 100, 250, 500 | E, SL0, SL2, SL5, CVAR90, CVAR95 | SMIP, BDP, BCP, STS, STSFG, STSFGBJ | 7.2, 7.3 |
| Large, one random job | DP 01a–18a (18), Hurink mt06, mt10, mt20, la01, la06, la11, la16, la21, la26, la31, la36 in three versions (33) | one job, each job in turn | 500 | E, SL0, CVAR90 | STS, STSFG, STSFGBJ | 7.2.2, 7.3 |
| Large, all random | as above | all | 500 | E, SL0, CVAR90 | STS, STSFG, STSFGBJ, BCP | 7.2, 7.3 |
| Sample size | Kacem3, Fattahi5, 10, 13, 15 | all | 50, 100, 250, 500, 1000; 10 independent samples each | E, SL0, CVAR90 | BCP, STS | 7.4 |
| Deadline | Fattahi13, 16, 20, Hurink edata5 (la02), rdata6 (la03), vdata7 (la04) | all | 500 | SL with δ ∈ {−10, −5, 0, 2, 5, 10, 15, 20}% | STS | 7.5 |

Settings:
- `E`: expected makespan;
- `SL0`, `SL2`, `SL5`: service level for the deadline T = round((1 + δ/100) · Cmax(s0)) with δ = 0, 2, 5%, where Cmax(s0) is given in `Inputs/deadlines.csv`;
- `CVAR90`, `CVAR95`: CVaR_β with β = 0.90, 0.95.

A case is an instance (with its random job in the one-job setting), a sample size and a setting. All methods of a case use the same scenarios. The out-of-sample set has 5000 scenarios (10 000 common scenarios in the sample-size experiment).

## Methods

| Name | Method |
|---|---|
| `SMIP` | extensive form of the SAA problem, solved by Gurobi |
| `BDP` | iterative Benders decomposition (master problem kept in memory, 300 s per master problem) |
| `BCP` | branch-and-check variant of the Benders decomposition (cuts added as lazy constraints) |
| `STS` | the proposed stochastic tabu search (10 000 iterations without improvement, at most five back jumps) |
| `STSFG` | the tabu search of Flores-Gómez, Borodin and Dauzère-Pérès (2023), and its adaptation to the CVaR |
| `STSFGBJ` | STSFG with the back jumps of STS |
| `s0` | the deterministic solution: tabu search of Dauzère-Pérès and Paulli (1997) on the nominal processing times, evaluated on the scenarios |

All runs have a time limit of 7200 s on 8 cores of an AMD EPYC 7552 processor; the exact methods have 32 GB of memory and Gurobi 11.0.0 with a soft memory limit of 28 GB.

## Scenarios

The processing time of operation i on machine m follows a beta distribution on [0.8μ, 1.8μ] with mean μ and standard deviation 0.15μ, where μ is the processing time in the benchmark instance (shape parameters a ≈ 1.22, b ≈ 4.89), rounded to the nearest integer with a minimum of one. In the one-job setting, only the operations of the random job are random and the other operations have their nominal times. The scenarios have equal probabilities. The in-sample samples use seed 1 (seeds 1–10 for the ten samples of the sample-size experiment), and samples of different sizes with the same seed are nested.

## Output files

All values are per run. `obj` is the in-sample value of the optimized measure (E[Cmax], α(T) or CVaR_β); `bound` is the bound of an exact method (a lower bound for E and CVaR, an upper bound for the service level); `time_s` is the wall-clock time in seconds; `in_*` and `oos_*` are the in-sample and out-of-sample values of the solution for the three measures (α at the deadline `T` of the run, which is the SL0 deadline Cmax(s0) for the E and CVaR settings except for some SMIP runs, which use the nominal makespan of their own warm start; CVaR with β = 0.9 except in the CVAR95 setting, where it is β = 0.95). `random_job` is the 0-based index of the random job (job 0 is job 1 in the text). `status` is `optimal`, `time limit` or `memory limit`, and `memory_stop` is 1 when the run stopped at the memory limit.

| File | Content |
|---|---|
| `Outputs/small_instances.csv` | 4536 runs: 756 cases × 6 methods. The out-of-sample values of BDP and BCP are not reported, since the comparisons of Section 7.3 use STS |
| `Outputs/large_one_random_job.csv` | 7047 runs: 2349 cases (783 instance–job pairs × 3 settings) × 3 tabu searches |
| `Outputs/large_all_random.csv` | 612 runs: 153 cases × 4 methods |
| `Outputs/deterministic_solution_s0.csv` | s0 evaluated in every setting of the three experiments above (value of the stochastic solution) |
| `Outputs/proved_optima_E_vs_SL0.csv` | the BCP solutions for E and SL0 of the small instances, both evaluated at the deadline T of SL0, with the indicator of proved optimality |
| `Outputs/paired_comparison_of_measures.csv` | paired comparison of the solutions optimized for E, SL0 and CVAR90 (see below) |
| `Outputs/makespan_distributions/` | out-of-sample makespan of the three STS solutions in each of the 5000 out-of-sample scenarios, for DP 01a with all operations random and for Hurink la11 (edata) with job 1 random (figure `makespan_cdfs.pdf`) |
| `Outputs/sample_size_replications.csv` | 1500 runs of the sample-size experiment (5 instances × 5 sizes × 3 settings × 10 samples × BCP, STS); `replication` is the seed of the sample |
| `Outputs/deadline.csv` | out-of-sample service level of the STS solution for the service level at each deadline (`STS_SL`), of the STS solution for the expectation (`STS_E`) and of s0 |

## Statistical analyses

`Outputs/paired_comparison_of_measures.csv` holds, for every case and pair of solutions x, y ∈ {E, SL0, CVAR90}, the differences of the out-of-sample values (d = x − y) with paired 95% confidence intervals. All solutions of a case are evaluated on the same 5000 out-of-sample scenarios.
- service level: paired Wald interval (±1.96 paired standard error) of the difference of the two proportions, with the exact McNemar test (`p_mcnemar`); `d_alpha` as a fraction;
- expected makespan: paired t-interval and test (`p_paired_t`) on the scenario makespans;
- CVaR_0.9: paired percentile bootstrap interval with 2000 resamples of the scenarios (the same resample for x and y), with the seed `bootstrap_seed` of the case (numpy `default_rng`).

`comparison` is `STS_solutions` for the STS solutions of all settings and `proved_optima_BCP` for the BCP optima on the small instances: E against SL0 on all cases, and E and SL0 against CVAR90 on the cases in which both optima of the pair are proved. `identical` is 1 when both solutions have the same assignment and sequences, and `correlation` is the Pearson correlation of their scenario makespans.

The workbook adds the per-group summaries of these intervals (`Paired_summary`), the paired t-tests between the tabu searches (`TS_ttests`), and the quantities of the SAA method (`SAA_summary`: in-sample bias, spread of the out-of-sample values, estimated optimality gap with respect to the mean of the SAA optimal values).

## Workbook

`results/SFJSP_Section7_results.xlsx` contains a `README` sheet that describes every sheet. The case sheets hold the results, and the statistics of the tables and of the text are worksheet formulas on them (Excel computes them when the file is opened). The sheets `Fig_TS` and `Fig_Exact` reproduce the figures of Section 7.2 as Excel charts, and the sheets `Paired_*`, `SAA_*`, `Deadline_*` and `Makespan_distributions` contain the data of Sections 7.3–7.5.

| Figure | Section | Data |
|---|---|---|
| `exact_overview.pdf` | 7.2.1 | `Small_cases`, `Fig_Exact` |
| `ts_overview.pdf` | 7.2.2 | `Small_cases`, `Large_job_cases`, `Large_all_cases`, `Fig_TS` |
| `makespan_cdfs.pdf` | 7.3 | `Makespan_distributions` |
| `saa_convergence.pdf` | 7.4 | `SAA_runs`, `SAA_summary` |
| `sl_curves.pdf` | 7.5 | `Deadline_runs`, `Deadline_summary` |

## Instance format

Instances are text files with literal section headers:
- `Jobs:` and `Machines:` (IDs);
- `Operations per job:` (one line per job);
- `Predecessors per operation:` and `Successors per operation:`;
- `Machines per operation:`;
- `Processing times (o m p):`, one row per eligible (operation, machine) pair with its nominal processing time.

All IDs are 0-based.

Instance names: `Fattahi1`–`10` = SFJS1–10, `Fattahi11`–`20` = MFJS1–10; `Hurink<v>data1` = mt06, `2` = mt10, `3` = mt20, `3+k` = la(k) (e.g. `HurinkEdata14` = la11, edata); `DPpaulli1`–`18` = 01a–18a.

## Benchmark sources

- **Kacem**: Kacem, I., Hammadi, S., Borne, P. (2002). Approach by localization and multiobjective evolutionary optimization for flexible job-shop scheduling problems. *IEEE Transactions on Systems, Man, and Cybernetics, Part C*, 32(1), 1–13.
- **Fattahi**: Fattahi, P., Saidi Mehrabad, M., Jolai, F. (2007). Mathematical modeling and heuristic approaches to flexible job shop scheduling problems. *Journal of Intelligent Manufacturing*, 18(3), 331–342.
- **Hurink**: Hurink, J., Jurisch, B., Thole, M. (1994). Tabu search for the job-shop scheduling problem with multi-purpose machines. *OR Spektrum*, 15(4), 205–215.
- **DP**: Dauzère-Pérès, S., Paulli, J. (1997). An integrated approach for modeling and solving the general multiprocessor job-shop scheduling problem using tabu search. *Annals of Operations Research*, 70, 281–306.
- **Scenario generation and STSFG**: Flores-Gómez, M., Borodin, V., Dauzère-Pérès, S. (2023). Maximizing the service level on the makespan in the stochastic flexible job-shop scheduling problem. *Computers & Operations Research*, 157, 106237.
