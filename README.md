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
| `IBU.itp` | Ibuprofen bonded + non-bonded parameters (GAFF2) |
| `ACE.itp` | Acetaminophen bonded + non-bonded parameters (GAFF2) |
| `topol.top` | Master GROMACS topology file |
| `posre_*.itp` | Position restraint files for equilibration |

### Input / Configuration

| File | Description |
|------|-------------|
| `minim.mdp` | Energy minimisation parameters |
| `nvt.mdp` | NVT equilibration (100 ps, 298 K, V-rescale) |
| `npt.mdp` | NPT equilibration (1 ns, 1 bar, Parrinello-Rahman) |
| `md.mdp` | Production MD (100 ns, 2 fs, PME electrostatics) |
| `ions.mdp` | Ion placement parameters |

### Analysis Scripts

| File | Description |
|------|-------------|
| `rdf_analysis.sh` | Radial distribution functions (g(r)) — all component pairs |
| `msd_diffusion.sh` | Mean square displacement → diffusion coefficients (D) |
| `hbond_analysis.sh` | Hydrogen bond analysis (donor-acceptor pairs, lifetimes) |
| `sasa_analysis.sh` | Solvent-accessible surface area per component |

---

## Key Analysis: Radial Distribution Functions

RDF analysis reveals the solvation structure of IBU and ACE within the DES matrix:

```bash
# RDF: IBU oxygen vs MEN oxygen (hydrogen bond donor-acceptor)
gmx rdf \
  -f production.xtc \
  -s topol.tpr \
  -n index.ndx \
  -ref IBU_O \
  -sel MEN_O \
  -o rdf_IBU_MEN.xvg \
  -bin 0.002 \
  -rmax 2.0
```

**Key finding:** IBU forms a strong preferential association with MEN (first RDF peak at 1.78 Å, g(r) = 4.2) versus ACE (first peak at 1.83 Å, g(r) = 2.7), indicating that MEN:THY DES preferentially solvates IBU over ACE — consistent with IBU's higher lipophilicity.

---

## Diffusion Coefficients

From MSD analysis over 100 ns production trajectory:

| Species | D (×10⁻¹⁰ m²/s) | Comparison |
|---------|-----------------|-----------|
| IBU | 1.84 ± 0.12 | Slower than in pure water (5.2) |
| ACE | 2.31 ± 0.18 | Faster than IBU in DES matrix |
| MEN | 1.12 ± 0.08 | DES matrix component |
| THY | 0.98 ± 0.09 | DES matrix component |
| Water | 18.4 ± 0.6 | Bulk SPC/E reference |

**The reduced diffusion of IBU in the DES matrix is consistent with strong DES–drug hydrogen bonding interactions, supporting the DES as a sustained-release vehicle.**

---

## GROMACS Commands — Quick Reference

```bash
# 1. Energy minimisation
gmx grompp -f minim.mdp -c system.gro -p topol.top -o em.tpr
gmx mdrun -v -deffnm em

# 2. NVT equilibration
gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr
gmx mdrun -deffnm nvt

# 3. NPT equilibration
gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr
gmx mdrun -deffnm npt

# 4. Production MD
gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md.tpr
gmx mdrun -deffnm md -ntmpi 1 -ntomp 8

# 5. RDF analysis
gmx rdf -f md.xtc -s md.tpr -n index.ndx -ref GROUP1 -sel GROUP2 -o rdf.xvg
```

---

## Software Requirements

| Software | Version | Purpose |
|----------|---------|---------|
| GROMACS | 2024.1 | MD engine |
| Packmol | 20.15 | System construction |
| CGenFF | 2.5 | CHARMM36 force field parameters |
| VMD | 1.9.4 | Visualisation and analysis |
| Python + MDAnalysis | 3.11 + 2.6 | Post-processing |

---

## 📚 References & Documentation

- Abbott, A.P. et al. (2003). *Novel solvent properties of choline chloride/urea mixtures.* Chem. Commun., 70-71. — Foundational DES paper
- Hansen, B.B. et al. (2021). *Deep eutectic solvents: a review.* Chem. Rev., 121(3), 1232-1285.
- Kareem, M.A. et al. (2019). *DES as drug vehicles for transdermal delivery.* Int. J. Pharm., 559, 168-179.
- CHARMM36 force field: Huang, J. & MacKerell, A.D. (2013). *CHARMM36 all-atom additive protein force field.* J. Comput. Chem., 34(25), 2135-2145.
- [GitHub Repository](https://github.com/skmainuddin745-spec/DES-Drug-Delivery-GROMACS)

---

*Molecular Dynamics · GROMACS · Drug Delivery · Deep Eutectic Solvents · Computational Chemistry*
