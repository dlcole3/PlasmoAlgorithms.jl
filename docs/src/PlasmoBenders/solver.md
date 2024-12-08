# Solver Options
PlasmoBenders supports several different solver options. These are keyword arguments that can be  These include the following: 
 * `strengthened::Bool` - whether to use "strengthened" Benders cuts for MILP problems. These cuts are outlined [here](https://link.springer.com/article/10.1007/s10107-018-1249-5) and [here](https://www.sciencedirect.com/science/article/pii/S0377221718304466). They add some computational time to the algorithm but can result in tighter cuts (and thus fewer iterations or a smaller duality gap) being added to the subproblems. This only applies if there are integer variables in the subproblems (not just the root subgraph). Currently, these only work for MILP problems, not for LP problems.
 * `multicut::Bool` - Whether to use multi-cuts (instead of aggregated cuts). When a subproblem has multiple children subgraphs (i.e., is connected to multiple subproblems in the next stage), you can use either an aggregated cut (where there is a single cost-to-go variable) or multi-cuts, where each child subproblem has its own corresponding cost-to-go function. Multi-cuts result in more cutting planes being added at each iteration, but aggregated cuts result in fewer overall constraints being added. 
 * `regularize::Bool` - Whether to use a regularization scheme to choose the solutions passed to the next stage. PlasmoBenders implements the regularization scheme from [here](https://arxiv.org/abs/2403.02559). The regularization only works for Benders Decomposition (BD; i.e., 2-stage problems). This scheme chooses points that are normally interior, feasible points in the master problem. Does not work with `warm_start=true`.
 * `regularize_param::Real` - A parameter between 0 and 1 that influences how far inside the feasible region the regularization can choose a solution.
 * `parallelize_benders::Bool` - For BD, whether to parallelize the solution of the second stage subgraphs if multiple subgraphs exist.
 * `parallelize_forward::Bool` - Not currently supported. We hope to implement this functionality in the future
 * `parallelize_backward::Bool` - Whether to parallelize the backwards pass.
 * `add_slacks::Bool` - Whether to add slack variables to the linking constraint that is enforced on the downstream problem. This helps ensure recourse to the subproblems.
 * `slack_penalty::Real` - THe value of the penalty term used on the slack variables added to the linking constraints. 
 * `fix_slacks::Bool` - An experimental option where slack variables are fixed until a problem is infeasible, in which case, the slack variables are allowed to be unfixed to make the problem feasible. THere are no convergence guarantees on this approach.
 * `warm_start::Bool` - Whether to warm start the problems with the best solution that the solver has found so far. Cannot be true if `regularize=true`. 
 * `relaxed_init_cuts::Bool` - Whether to add initial cuts from a full problem relaxation. For MILP problems, initial cuts can be added by relaxing all integer variables and solving the full problem, and using the resulting primal/dual information to form cutting planes. See [Lara](https://www.sciencedirect.com/science/article/pii/S0377221718304466).
 * `max_iters::Int` - The maximum number of iterations to use before termination.
 * `tol::Real` - The relative tolerance between the upper and lower bounds to reach before termination. 
 * `solver` - The subproblem solver to use. If this is not set, it will assume that the user has set a solver on all subgraphs. 
 * `M::Real` - Lower bound on all cost-to-go variables. Default value is $0$. At the first iteration of the BD and Nested Benders Decomposition, theta is otherwise unconstrained, so this value ensures that the cost-to-variable is bounded. 
 * `is_MIP::Bool` - Whether the problem includes mixed integer variables in the subproblems. If the user does not set this value, the `BendersOptimizer` constructor will detect this value by testing all subgraphs that are not the root subgraph (note that the root subgraph can have mixed integer variables and still set `is_MIP` to `false`). This argument determines when the backward pass occurs. For LPs, the backward pass can occur at the same time as the forward pass, but for MILPs, the backward pass is performed separately (and can be parallelized). 
 
!!! note
    If the objective value of a subproblem can be negative, it is important to set `M` to be less than zero. Otherwise, you can get lower bounds that are greater than upper bounds and the algorithm will not work. 

The `JuMP.optimize!` function has been extended to solve a `BendersOptimizer` object. There are two additional keyword arguments you can set with `JuMP.optimize!`
 * `output::Bool` - Whether to output the upper and lower bounds and the gap at each iteration of the algorithm
 * `run_gc::Bool` - Whether to run the garbage collector at the end of each iteration. Could potentially help with some memory issues. 
    
## Reporting Issues
If you encounter issues with PlasmoBenders, please open issues on Github's [issue tracker](https://github.com/plasmo-dev/PlasmoAlgorithms.jl/issues). 