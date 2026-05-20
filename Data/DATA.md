# Data

## Sources

* **Provider:** ATLAS Collaboration, CERN
* **Release:** ATLAS Open Data 2025, tag `2025e-13tev-beta`
* **Collision data:** pp at √s = 13 TeV, 2015–2016, ∫L = 36.6 fb⁻¹
* **Monte Carlo:** signal H → ZZ\* → 4ℓ + ZZ\* / Z+jets / tt̄ / tt̄+V / VVV backgrounds
* **Format:** ROOT trees
* **Skim:** `exactly4lep` (events with ≥ 4 reconstructed leptons)
* **Portal:** <https://opendata.atlas.cern>
* **Spec:** <https://opendata.atlas.cern/docs/data/for_education/13TeV25_details>

## Obtaining the Data

Files are not redistributed with this project. They are streamed at runtime
from the ATLAS Open Data servers via the `atlasopenmagic` helper, which caches
each file locally on first access.

```bash
pip install uproot awkward vector atlasopenmagic
```

```python
import atlasopenmagic as atom
atom.set_release('2025e-13tev-beta')

defs = {
    'Data':                                       {'dids': ['data']},
    'Background Z,ttbar,ttbar+V,VVV':             {'dids': [410470, 410155, 410218, 410219, 412043,
                                                             364243, 364242, 364246, 364248,
                                                             700320, 700321, 700322, 700323,
                                                             700324, 700325]},
    'Background ZZ*':                             {'dids': [700600]},
    'Signal (mH = 125 GeV)':                      {'dids': [345060, 346228, 346310, 346311, 346312,
                                                             346340, 346341, 346342]},
}
samples = atom.build_dataset(defs, skim='exactly4lep', protocol='https', cache=True)
```

Manual fallback: browse <https://opendata.atlas.cern/release/2025/>, download
the ROOT files for the dataset IDs (DIDs) listed above, then point
`uproot.open(...)` at the local paths.

## Datasets Used

| Group                        | Dataset IDs                                                                                       | Physical process                          |
| ---------------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| Data                         | `data`                                                                                            | 2015–2016 ATLAS pp collisions, 36.6 fb⁻¹  |
| Signal (mH = 125 GeV)        | 345060, 346228, 346310–312, 346340–342                                                            | H → ZZ\* → 4ℓ, all production modes        |
| Bkg — ZZ\* (irreducible)     | 700600                                                                                            | qq → ZZ\* → 4ℓ continuum                   |
| Bkg — Z, tt̄, tt̄+V, VVV     | 410470, 410155, 410218, 410219, 412043, 364242–248, 700320–325                                    | Reducible 4-lepton backgrounds            |

## Preprocessing

Implemented in `HZZAnalysis_clean.ipynb`, sections 3 and 5.

1. **Trigger:** require `trigE | trigM` and ≥ 1 trigger-matched lepton.
2. **Lepton ID/iso:** electrons must pass `lep_isLooseID & lep_isLooseIso`;
   muons must pass `lep_isMediumID & lep_isLooseIso`; required for all 4 leptons.
3. **SFOS pairing:** keep events with lepton-type sum ∈ {44, 50, 52} excluded
   (kept set: 4e / 4µ / 2e2µ) and charge sum = 0.
4. **pT staircase:** ordered leptons must satisfy pT > (20, 15, 10, 7) GeV.
5. **Z₁/Z₂ reconstruction:** among the three possible SFOS pairings, pick the
   one whose dilepton mass is closest to MZ = 91.2 GeV → on-shell Z₁; the
   remaining pair → off-shell Z₂. Events with no valid pairing are dropped.
6. **m_4ℓ:** invariant mass of the summed four-lepton 4-momentum, computed
   with the `vector` library.
7. **MC event weighting** (paper Eqs. 1–2):

   ```
   w_total = (L · σ · η · k · w_MC / Σw_i)
           · SF_pileup · SF_electron · SF_muon · SF_trigger
   ```

   where L = 36.6 fb⁻¹, σ = `xsec`, η = `filteff`, k = `kfac`,
   w_MC = `mcWeight`, Σw_i = `sum_of_weights`. Real data carry weight 1.
8. **Feature engineering** (paper §VII.A): 31 physics-motivated features built
   from the 4-vectors (Z masses & pT, 4ℓ pT, per-lepton pT/η after Z
   assignment, 5 helicity-frame angles, intra-Z Δφ, cross-pair ΔR, scalar pT
   sums, jet count, MET, ϕ_miss).
9. **Feature selection** (paper §VII.B): separation power δ² on the training
   set, then prune features with Pearson |ρ| ≥ 0.7 in signal → 20 features
   retained (`keep_sep`).

### Yields after full preselection (Table II)

| Process                       | Raw events | Weighted yield (36.6 fb⁻¹) |
| ----------------------------- | ---------- | -------------------------- |
| Signal H → ZZ\* → 4ℓ          | 226,168    | 30.43 ± 1.33               |
| Bkg ZZ\* (irreducible)        | 6,364      | 1067.08 ± 18.15            |
| Bkg reducible                 | 6,033      | 127.61 ± 16.32             |
| **Total background**          | **12,436** | **1194.69 ± 24.41**        |
| **Real data**                 | **1,279**  | —                          |

## Variable Descriptions

### Raw ROOT branches

| Branch                     | Type    | Units      | Description                                                |
| -------------------------- | ------- | ---------- | ---------------------------------------------------------- |
| `lep_pt[4]`                | float[] | GeV        | Lepton transverse momenta.                                 |
| `lep_eta[4]`               | float[] | —          | Pseudorapidities η = −ln tan(θ/2).                         |
| `lep_phi[4]`               | float[] | rad        | Lab-frame azimuthal angles, (−π, π].                       |
| `lep_e[4]`                 | float[] | GeV        | Lepton energies.                                           |
| `lep_charge[4]`            | int[]   | ±1         | Lepton electric charges.                                   |
| `lep_type[4]`              | int[]   | 11 / 13    | PDG ID: 11 = electron, 13 = muon.                          |
| `lep_isLooseID`            | bool[]  | T/F        | Loose ID (used for electrons).                             |
| `lep_isMediumID`           | bool[]  | T/F        | Medium ID (used for muons).                                |
| `lep_isLooseIso`           | bool[]  | T/F        | Loose isolation.                                           |
| `lep_isTrigMatched`        | bool[]  | T/F        | Lepton matched to a trigger object.                        |
| `trigE`, `trigM`           | bool    | T/F        | Single-e / single-µ trigger decisions.                     |
| `lep_n`                    | int     | ≥ 4        | Number of reconstructed leptons.                           |
| `jet_n`                    | int     | ≥ 0        | Number of reconstructed jets.                              |
| `met`                      | float   | GeV        | Missing transverse energy magnitude.                       |
| `met_phi`                  | float   | rad        | MET azimuthal angle.                                       |
| `mcWeight`                 | float   | —          | Generator-level event weight (MC).                         |
| `xsec`                     | float   | pb         | Process cross-section (MC).                                |
| `filteff`                  | float   | —          | Generator filter efficiency (MC).                          |
| `kfac`                     | float   | —          | Higher-order correction factor (MC).                       |
| `sum_of_weights`           | float   | —          | Sum of generator weights for the sample (MC).              |
| `ScaleFactor_PILEUP`       | float   | ≈ 1        | Pile-up reweighting SF (MC).                               |
| `ScaleFactor_ELE`          | float   | ≈ 1        | Electron reconstruction / ID SF (MC).                      |
| `ScaleFactor_MUON`         | float   | ≈ 1        | Muon reconstruction / ID SF (MC).                          |
| `ScaleFactor_LepTRIGGER`   | float   | ≈ 1        | Lepton trigger SF (MC).                                    |

### Derived per-event quantities

| Variable        | Units | Description                                                         |
| --------------- | ----- | ------------------------------------------------------------------- |
| `mass` (`m_4ℓ`) | GeV   | Four-lepton invariant mass; main discriminant.                       |
| `totalWeight`   | —     | Luminosity-scaled event weight w_total (MC only).                   |

### Engineered classifier features (31 candidates → 20 retained)

| Feature                                                   | Units   | Description                                                                                          |
| --------------------------------------------------------- | ------- | ---------------------------------------------------------------------------------------------------- |
| `mz1`, `mz2`                                              | GeV     | Invariant masses of the on-shell (Z₁) and off-shell (Z₂) Z candidates.                               |
| `ptz1`, `ptz2`                                            | GeV     | Vector-sum pT of Z₁, Z₂.                                                                             |
| `pt4l`                                                    | GeV     | Magnitude of the 4-lepton vector-sum pT.                                                             |
| `pt_z1_l1`, `pt_z1_l2`, `pt_z2_l1`, `pt_z2_l2`            | GeV     | Lepton pT after Z assignment, ordered (lead, sub) within each Z.                                     |
| `eta_z1_l1`, `eta_z1_l2`, `eta_z2_l1`, `eta_z2_l2`        | —       | Lepton η after Z assignment, same ordering as above.                                                 |
| `theta1`, `theta2`                                        | rad     | Angle between the boosted negative lepton and −p̂_H in each Z's rest frame.                          |
| `Theta`                                                   | rad     | Angle between the boosted Z₁ direction and the beam axis in the Higgs rest frame.                    |
| `Phi`                                                     | rad     | Signed angle between the Z₁ and Z₂ decay planes in the Higgs rest frame, ∈ [−π, π].                  |
| `Phi1`                                                    | rad     | Signed angle between the Z₁ decay plane and the (p⃗_Z₁, beam) plane, ∈ [−π, π].                       |
| `dphi_z1`, `dphi_z2`                                      | rad     | Intra-Z Δφ between the two leptons of Z₁ / Z₂, wrapped to [0, π].                                    |
| `dR_z1`, `dR_z2`                                          | —       | Intra-Z ΔR = √(Δη² + Δφ²) for Z₁ / Z₂.                                                               |
| `dR_02`, `dR_03`, `dR_12`, `dR_13`                        | —       | Cross-pair ΔR between (Z₁ lead/sub) and (Z₂ lead/sub) leptons.                                       |
| `scalar_pt_sum_z1`, `scalar_pt_sum_z2`                    | GeV     | Arithmetic pT sum of each Z's two leptons (independent of decay-plane orientation).                  |
| `jet_n`, `met`, `met_phi`                                 | —, GeV, rad | Event-level jet multiplicity and missing-transverse-energy variables.                            |
| `type_z1_l1`, `type_z1_l2`, `type_z2_l1`, `type_z2_l2`    | 11 / 13 | Lepton-flavour bookkeeping (plotting only, not passed to classifiers).                               |
