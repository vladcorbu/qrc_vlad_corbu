# Getting Started with Bloqade in Julia - A Tutorial

## Installation

First, let's install Bloqade in Julia:

```julia
using Pkg
Pkg.add("Bloqade")
```

## Basic Usage

```julia
using Bloqade
```

## Creating a Rydberg Atom Chain

```julia
# Create a 1D lattice with 5 atoms
n_atoms = 5
atoms = generate_sites(ChainLattice(), n_atoms, scale=5.0)

# Create a basic Hamiltonian
h = rydberg_h(atoms)
```

## Setting Up the Rabi Coupling

```julia
# Define the Rabi frequency profile
Ω_max = 2π * 4.0  # MHz
T = 1.0           # μs

Ω(t) = Ω_max * sin(π * t / T)^2
```

## Adding Terms to the Hamiltonian

```julia
# Add the Rabi coupling term
h += rabi(Ω)

# Add detuning term
Δ(t) = -2π * 10.0 * (1 - 2t/T)
h += detuning(Δ)
```

## Running a Simulation

```julia
# Define initial state (all atoms in ground state)
ψ₀ = zero_state(n_atoms)

# Set up the simulation problem
tspan = (0.0, T)
prob = SchrodingerProblem(ψ₀, tspan, h)

# Run the simulation
sol = solve(prob)
```

## Analyzing Results

```julia
# Calculate Rydberg populations
times = range(0, T, length=100)
populations = rydberg_density(sol, times)

# Calculate correlation functions
correlations = correlation_matrix(sol, times[end])
```

## Visualization Example

```julia
using Plots

# Plot Rydberg populations over time
plot(times, populations,
     xlabel="Time (μs)",
     ylabel="Rydberg Population",
     label=["Atom $i" for i in 1:n_atoms])

# Plot correlation matrix
heatmap(correlations,
        xlabel="Atom i",
        ylabel="Atom j",
        title="Rydberg Correlations")
```

## Advanced Features

### Creating Custom Pulse Sequences

```julia
# Define a piecewise pulse sequence
Ω_piece(t) = piecewise_linear(t, [0.0, T/2, T], [0.0, Ω_max, 0.0])
Δ_piece(t) = piecewise_constant(t, [0.0, T/2, T], [0.0, -2π * 10.0, 0.0])

h_custom = rydberg_h(atoms) + rabi(Ω_piece) + detuning(Δ_piece)
```

### Parallel Execution

```julia
using Distributed
addprocs(4)  # Add worker processes

# Run parallel simulations
@everywhere using Bloqade
results = pmap(1:10) do i
    prob = SchrodingerProblem(ψ₀, tspan, h)
    solve(prob)
end
```

## Best Practices

1. Always check your units (Bloqade uses MHz and μs by default)
2. Monitor conservation laws during simulation
3. Use appropriate numerical tolerances
4. Save intermediate results for long simulations

```julia
# Example of saving intermediate results
prob = SchrodingerProblem(ψ₀, tspan, h)
sol = solve(prob, saveat=range(0, T, length=100))
```

## Error Handling

```julia
try
    sol = solve(prob)
catch e
    if isa(e, DimensionMismatch)
        println("Check your Hilbert space dimensions")
    else
        rethrow(e)
    end
end
```

Remember to:
- Always validate your geometry before running long simulations
- Check energy scales are compatible with your timesteps
- Verify your results with known physical constraints

