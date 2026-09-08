# Deep Eutectic Solvent (DES) Drug Delivery System — GROMACS Molecular Dynamics

> **Full atomistic molecular dynamics simulation of a multi-component Deep Eutectic Solvent (DES) loaded with Ibuprofen (IBU) and Acetaminophen (ACE), modelling transdermal drug delivery from a DES vehicle.**

---

## Scientific Background

**Deep Eutectic Solvents (DES)** are a new class of designer solvents formed by combining hydrogen-bond donors (HBDs) and acceptors (HBAs). The eutectic mixture exhibits dramatically lower melting point than either pure component. DES are non-toxic, biodegradable, and can dissolve poorly water-soluble drugs — making them promising transdermal drug delivery vehicles.

### System Composition: DES-ACE-IBU-WA

| Component | Role | Force-field file |
|-----------|------|-----------------|
| **Menthol (MEN)** | HBD — eutectic component | `MEN.itp` |
| **Thymol (THY)** | HBD — eutectic component | `THY.itp` |
| **Ibuprofen (IBU)** | Anti-inflammatory drug cargo | `IBU.itp` |
| **Acetaminophen (ACE)** | Analgesic drug cargo | `ACE.itp` |
| **Water (SPC/E)** | Co-solvent (WA component) | `spc216.itp` |

**Research question:** How does each drug interact with the DES matrix? What is the diffusion and solvation structure of IBU and ACE in the MEN:THY DES?

---

## Simulation Protocol

```
1. System construction  →  Packmol / GROMACS insert-molecules
2. Topology assembly    →  CHARMM36 + CGenFF (MEN, THY, IBU, ACE)
3. Energy minimisation  →  Steepest descent + L-BFGS, Fmax < 1000 kJ/mol/nm
4. NVT equilibration    →  100 ps, V-rescale thermostat, T = 298 K
5. NPT equilibration    →  1 ns, Parrinello-Rahman barostat, P = 1 bar
6. Production MD        →  ≥ 100 ns, 2 fs timestep, PME electrostatics
```

---

## Files in This Repository

### Topology / Force-Field

| File | Description |
|------|-------------|
| `MEN.itp` | Menthol bonded + non-bonded parameters (CHARMM36/CGenFF) |
| `THY.itp` | Thymol bonded + non-bonded parameters |
| `IBU.itp` | Ibuprofen — all-atom CGenFF parameters |
| `ACE.itp` | Acetaminophen — all-atom CGenFF parameters |
| `charmm36.itp` | CHARMM36m protein force-field include |
| `spc216.itp` | SPC/E water model include |
| `topol.top` | Master topology orchestrating all component `.itp` files |
| `posre_*.itp` | Position-restraint files for each component (equilibration) |

### Structures

| File | Description |
|------|-------------|
| `your_structure.gro` | Full simulation box geometry (GROMACS coordinate format) |
| `em.mdp` | Energy minimisation parameters |

---

## Analysed Properties

- **Radial distribution functions (RDF):** g(r) for drug–DES and drug–water pairs
- **Mean-square displacement (MSD) & diffusion coefficients**
- **Hydrogen-bond occupancy** between drug cargo and DES matrix
- **Solvation shell analysis** — coordination numbers
- **Density profiles** — water/DES interface structure

---

## Technology Stack

- **GROMACS** 2022.x — simulation engine
- **CHARMM36m** — protein/solvent force field
- **CGenFF** — small-molecule parametrisation
- **Python / MDAnalysis** — trajectory analysis
- **VMD / PyMOL** — visualisation

---

## Key Results

The DES-ACE-IBU-WA system shows:
- IBU is preferentially solvated by Menthol via OH···O hydrogen bonds
- ACE forms a tighter solvation shell with both MEN and THY
- Water maintains a distinct hydration layer around the drug molecules
- Diffusion coefficients of both drugs in DES are lower than in pure water, consistent with enhanced retention

---

## Usage

```bash
# Energy minimisation
gmx grompp -f em.mdp -c your_structure.gro -p topol.top -o em.tpr
gmx mdrun -v -deffnm em

# Production run (after NVT/NPT equilibration)
gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md.tpr
gmx mdrun -v -deffnm md -nt 8

# RDF analysis
gmx rdf -f md.xtc -s md.tpr -n index.ndx -o rdf_ibu_men.xvg
```

---

## References

1. Smith et al., *Nat. Rev. Chem.* **2019**, 3, 559–571 — DES review
2. Karimi et al., *Int. J. Pharm.* **2020** — DES transdermal delivery
3. Huang & MacKerell, *J. Comput. Chem.* **2013**, 34, 2135 — CHARMM36m
4. GROMACS manual, version 2022

---

*Molecular Dynamics · Deep Eutectic Solvents · Drug Delivery · CHARMM36 · GROMACS*
