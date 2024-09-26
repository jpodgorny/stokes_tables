Table of contents
=================

* [Description of STOKES tables and models](#description-of-STOKES-tables-and-models)
* [References](#references)
* [Parameter definitions for STOKES tables and models](#parameter-definitions-for-STOKES-tables-and-models)
* [Required files](#required-files)
* [Usage in XSPEC](#usage-in-xspec)
* [Viewing the STOKES tables and models in XSPEC](#viewing-the-STOKES-tables-and-models-in-xspec)


Description of STOKES tables and models
=======================================

The STOKES tables and models (Podgorný et al. 2022, Podgorný 2023) provide spectra
and polarisation properties of reprocessed emission in a plane-parallel slab 
illuminated by an X-ray source emitting power-law radiation. The incident photons 
may have arbitrary polarisation. The reprocessing is precomputed and stored in FITS 
files, which are calculated for three different states of incident polarisation. The 
tables include the dependence on the geometry of scattering defined by the incident, 
emission and azimuthal angles, θ<sub>i</sub>, θ<sub>e</sub> and φ. The slab is 
assumed to be optically thick, with a constant density of 
n<sub>H</sub>=10<sup>15</sup>cm<sup>-3</sup>. The power-law illumination is
characterised by a photon index, Γ, with sharp 
low- and high-energy cut-offs at E<sub>l</sub> ≈ 0.08 keV and E<sub>c</sub> ≈ 250 
keV, causing ionisation of the slab, which is defined by the ionisation parameter, 
ξ. The ionisation structure of the slab was computed using the TITAN code (Dumont et
al. 2003), while the spectral shape and polarisation properties of the reprocessed 
emission were computed using the STOKES code (Goosmann & Gaskell 2007, Marin 2018).

The provided tables conform to OGIP standards and can be directly used in XSPEC
using the `atable` command. Four FITS tables are available for the Stoke parameters 
i, q and u with 300 bins in 0.1 to 100 keV (the reduced version of these tables with 100 bins in 1 to 10 keV is also available at [stokes_tables_reduced-v2.tar.gz](https://owncloud.asu.cas.cz/index.php/s/qOcBk05jPV4bQNR)):

* [stokes_unpol_iso-v2.fits](https://owncloud.asu.cas.cz/index.php/s/lG7R3Ns5gDeDMkS)
→ for unpolarised isotropic illumination of the slab, i.e. the result is integrated 
over the incident and azimuthal angles, θ<sub>i</sub> and φ; the parameters of these 
tables include Γ, ξ and μ<sub>e</sub>,

* [stokes_unpol-v2.fits](https://owncloud.asu.cas.cz/index.php/s/dtShnA2HYb6lvdR)
→ for unpolarised illumination; the parameters of these tables include Γ, ξ,
μ<sub>i</sub>, μ<sub>e</sub> and φ,

* [stokes_vrpol-v2.fits](https://owncloud.asu.cas.cz/index.php/s/06rK1zxDJMhb4CV)
→ for fully polarised illumination in the vertical direction; the parameters of 
these tables include Γ, ξ, μ<sub>i</sub>, μ<sub>e</sub> and φ,

* [stokes_45deg-v2.fits](https://owncloud.asu.cas.cz/index.php/s/BH6GMtbJUanNlB8)
→ for fully polarised illumination with a polarisation angle of 45° from the 
vertical direction, measured counterclockwise; the parameters of 
these tables include Γ, ξ, μ<sub>i</sub>, μ<sub>e</sub> and φ,

where μ<sub>i</sub>=cos θ<sub>i</sub> and μ<sub>e</sub>=cos θ<sub>e</sub>.

Additionally, several STOKES models, which use the above tables, are provided for 
convenience. These models i) use the emission angle, θ<sub>e</sub>, instead of its 
cosine, μ<sub>e</sub>, and ii) allow for arbitrary polarisation fraction and 
polarisation direction of the illumination. All of them utilise the `mdefine` 
command in XSPEC. **Note that the `mdefine` command does not work for polarisation 
models in XSPEC versions 12.14.1b and earlier. To use the provided models, please, 
update your XSPEC to a later version!** For older versions, you may try this [workaround](#workaround-for-xspec-versions-12141b-and-earlier).

The following models are available:

* **`stiso`** → for unpolarised isotropic illumination; this is simply a 
redefinition of the `stokes_unpol_iso-v2.fits` table to use the emission angle 
θ<sub>e</sub> instead of its cosine, μ<sub>e</sub>, by utilising the `mdefine` 
command in XSPEC:  
`mdefine stiso atable{stokes_unpol_iso-v2.fits}(PhoIndex, Xi, cosd(Thetae), z) : add`  
thus the parameters of this model include Γ, ξ and θ<sub>e</sub>,

* **`stunp`**, **`stvrp`** and **`st45d`** → redefinitions of the
`stokes_unpol-v2.fits`, `stokes_vrpol-v2.fits` and `stokes_45deg-v2.fits` tables, 
respectively, to use the incident and emission angles, θ<sub>i</sub> and  θ<sub>e</sub>, instead of their cosines, μ<sub>i</sub> and μ<sub>e</sub>; the parameters of these models include Γ, ξ, θ<sub>i</sub>, θ<sub>e</sub> and φ,

* **`stpol`** → for illumination polarised in vertical or horizontal directions with 
any polarisation fraction; the polarisation fraction, P, for the vertical
polarisation direction is positive, while for the horizontal direction, it is 
negative; this model combines the models for unpolarised illumination, `stunp`, and 
fully vertcially polarised illumination, `stvrp`, according to the equation:  
S(P) = S(0, -) + P [ S(100%, 0°) - S(0, -) ], i.e.,  
`stpol` = `stunp` + P * [ `stvrp` - `stunp` ];  
the parameters of this model include Γ, ξ, θ<sub>i</sub>, θ<sub>e</sub>, φ and P, 
with -1 <= P <= 1,

* **`stokes`** → for illumination polarised in any direction, χ, with any 
polarisation fraction, P; this model combines the models for unpolarised 
illumination, `stunp`, fully vertically polarised illumination, `stvrp`, and fully polarised illumination at 45° counterclockwise from the vertical direction, `st45d`, 
according to the equation:  
S(P, χ) = S(0, -) + P { [ S(100%, 0°) - S(0, -) ] cos 2χ + [ S(100%, 45°) - S(0, -) ] sin 2χ }, i.e.,  
`stokes` = `stunp` + P * { [ `stvrp` - `stunp` ] * cos 2χ + [ `st45d` - `stunp` ] * sin 2χ };  
the parameters of this model include Γ, ξ, θ<sub>i</sub>, θ<sub>e</sub>, φ, P and χ,
with 0 <= P <= 1 and -90° <= χ <= 90°.

Note that the XSPEC mixed model for the rotation of the Stokes parameters,
**`polrot`**, must be used in combination with the above models, e.g.
**`polrot*stokes`**, to be able to fit for a specific orientation angle of the
system.

For any issues regarding the use of the STOKES tables and models, please, contact J.
Podgorný at [jakub.podgorny@asu.cas.cz](mailto:jakub.podgorny@asu.cas.cz) 
or M. Dovčiak at [michal.dovciak@asu.cas.cz](mailto:michal.dovciak@asu.cas.cz).


References
==========

Podgorný J, Dovčiak M, Marin F, Goosmann RW & Różańska A (2022)  
_Spectral and polarization properties of reflected X-ray emission from black hole accretion discs_  
[MNRAS, 510, pp.4723-4735](https://doi.org/10.1093/mnras/stab3714) 
[[arXiv:2201.07494](https://arxiv.org/abs/2201.07494)]

Podgorný J (2023)  
_Polarisation properties of X-ray emission from accreting supermassive black holes_  
[PhD thesis](https://ui.adsabs.harvard.edu/abs/2024arXiv240316746P) 
[[arXiv:2403.16746](https://arxiv.org/abs/2403.16746)]

Dumont AM, Collin S, Paletou F, Coupé S, Godet O & Pelat D (2003)  
_Escape probability methods versus ``exact" transfer for modelling the X-ray spectrum of Active Galactic Nuclei and X-ray binaries_  
[A&A, 407, p.13-30](https://doi.org/10.1051/0004-6361:20030890) 
[[arXiv:astro-ph/0306297](https://arxiv.org/abs/astro-ph/0306297)]

Goosmann RW & Gaskell CM (2007)  
_Modeling optical and UV polarization of AGNs. I. Imprints of individual scattering regions_  
[A&A, 465, pp.129-145](https://doi.org/10.1051/0004-6361:20053555)
[[arXiv:astro-ph/0507072](https://arxiv.org/abs/astro-ph/0507072)]

Marin F (2018)  
_Modeling optical and UV polarization of AGNs. V. Dilution by interstellar polarization and the host galaxy_  
[A&A, 615, id.A171](https://doi.org/10.1051/0004-6361/201833225) 
[[arXiv:1805.09098](https://arxiv.org/abs/1805.09098)]


Parameter definitions for STOKES tables and models
==================================================

`stokes_unpol_iso-v2.fits`
--------------------------

Tables for unpolarised isotropic illumination.

* **par1 ... Gamma** [ 1.4 <= Gamma <= 3.0 ]
  - photon index of the incident power-law X-ray flux
* **par2 ... Xi** [ 5. <= Xi <= 20 000. ]
  - ionisation parameter of the slab
* **par3 ... Mue** [ 0.025 <= Mue <= 0.975 ]
  - cosine of the emission angle (0.-disc, 1.-pole)

`stokes_unpol-v2.fits`, `stokes_vrpol-v2.fits` and `stokes_45ged-v2.fits`
-------------------------------------------------------------------------

Tables for unpolarised illumination, fully vertically polarised illumination and 
for fully polarised illumination with a polarisation angle of 45° counterclockwise 
from the vertical direction.

* **par1 ... Gamma** [ 1.4 <= Gamma <= 3.0 ]
  - photon index of the incident power-law X-ray flux
* **par2 ... Xi** [ 5. <= Xi <= 20 000. ]
  - ionisation parameter of the slab
* **par3 ... Mui** [ 0. <= Mui <= 1. ]
  - cosine of the incident angle (0.-disc, 1.-pole)
* **par4 ... Phi** [ 7.5 <= Phi <= 352.5 ]
  - azimuthal scattering angle in degrees (0.-forward, 180.-backward)
* **par5 ... Mue** [ 0.025 <= Mue <= 0.975 ]
  - cosine of the emission angle (0.-disc, 1.-pole)

`stiso`
-------

Model for unpolarised isotropic illumination.

* **par1 ... PhoIndex** [ 1.4 <= Gamma <= 3.0 ]
  - photon index of the incident power-law X-ray flux
* **par2 ... Xi** [ 5. <= Xi <= 20 000. ]
  - ionisation parameter of the slab
* **par3 ... Thetae** [ 12.839 <= Thetae <= 88.567 ]
  - emission angle in degrees (0°-pole, 90°-disc)
* **par4 ... zshift** [ 0. <= zshift <= 5. ]
  - overall Doppler shift

`stunp`, `stvrp` and `st45d`
----------------------------

Models for unpolarised illumination, fully vertically polarised illumination and 
for fully polarised illumination with a polarisation angle of 45° counterclockwise 
from the vertical direction.

* **par1 ... PhoIndex** [ 1.4 <= Gamma <= 3.0 ]
  - photon index of the incident power-law X-ray flux
* **par2 ... Xi** [ 5. <= Xi <= 20 000. ]
  - ionisation parameter of the slab
* **par3 ... Thetai** [ 0. <= Thetai <= 90. ]
  - incident angle in degrees (0°-pole, 90°-disc)
* **par4 ... Phi** [ 7.5 <= Phi <= 352.5 ]
  - azimuthal scattering angle in degrees (0.-forward, 180.-backward)
* **par5 ... Thetae** [ 12.839 <= Thetae <= 88.567 ]
  - emission angle in degrees (0°-pole, 90°-disc)
* **par6 ... zshift** [ 0. <= zshift <= 5. ]
  - overall Doppler shift

`stpol`
-------

Model for illumination polarised in vertical or horizontal directions with any polarisation fraction.

* **par1 ... PhoIndex** [ 1.4 <= Gamma <= 3.0 ]
  - photon index of the incident power-law X-ray flux
* **par2 ... Xi** [ 5. <= Xi <= 20 000. ]
  - ionisation parameter of the slab
* **par3 ... Thetai** [ 0. <= Thetai <= 90. ]
  - incident angle in degrees (0°-pole, 90°-disc)
* **par4 ... Phi** [ 7.5 <= Phi <= 352.5 ]
  - azimuthal scattering angle in degrees (0.-forward, 180.-backward)
* **par5 ... Thetae** [ 12.839 <= Thetae <= 88.567 ]
  - emission angle in degrees (0°-pole, 90°-disc)
* **par6 ... zshift** [ 0. <= zshift <= 5. ]
  - overall Doppler shift
* **par7 ... PolFrac** [ -1. <= PolFrac <= 1. ]
  - polarisation fraction of illumination  
  &lt; 0 for the horizontal polarisation direction  
  &gt; 0 for the vertical polarisation direction

`stokes`
--------

Model for illumination polarised in any direction with any polarisation fraction.

* **par1 ... PhoIndex** [ 1.4 <= Gamma <= 3.0 ]
  - photon index of the incident power-law X-ray flux
* **par2 ... Xi** [ 5. <= Xi <= 20 000. ]
  - ionisation parameter of the slab
* **par3 ... Thetai** [ 0. <= Thetai <= 90. ]
  - incident angle in degrees (0°-pole, 90°-disc)
* **par4 ... Phi** [ 7.5 <= Phi <= 352.5 ]
  - azimuthal scattering angle in degrees (0.-forward, 180.-backward)
* **par5 ... Thetae** [ 12.839 <= Thetae <= 88.567 ]
  - emission angle in degrees (0°-pole, 90°-disc)
* **par6 ... zshift** [ 0. <= zshift <= 5. ]
  - overall Doppler shift
* **par7 ... PolFrac** [ -1. <= PolFrac <= 1. ]
  - polarisation fraction of illumination  
  &lt; 0 for the horizontal polarisation direction  
  &gt; 0 for the vertical polarisation direction
* **par8 ... PolAng** [ -90. <= PolAng <= 90. ]
  - polarisation angle of illumination measured counterclockwise from vertical
    direction


Required files
==============

* **STOKES tables** with 300 bins in 0.1 to 100 keV:
  - [stokes_unpol_iso-v2.fits](https://owncloud.asu.cas.cz/index.php/s/lG7R3Ns5gDeDMkS)
  - [stokes_unpol-v2.fits](https://owncloud.asu.cas.cz/index.php/s/dtShnA2HYb6lvdR)
  - [stokes_vrpol-v2.fits](https://owncloud.asu.cas.cz/index.php/s/06rK1zxDJMhb4CV)
  - [stokes_45deg-v2.fits](https://owncloud.asu.cas.cz/index.php/s/BH6GMtbJUanNlB8)

* or **reduced STOKES tables** with 100 bins in 1 to 10 keV:
  - [stokes_tables_reduced-v2.tar.gz](https://owncloud.asu.cas.cz/index.php/s/qOcBk05jPV4bQNR)

* **package containing the fake null data and unit repsonses**
  - [fake_null_iqu_300ch.tar.gz](https://owncloud.asu.cas.cz/index.php/s/Flk6cwYLISmw0D5)

* **XSPEC scripts** with definition of the models
  - [stokes_tables-main.zip](https://github.com/jpodgorny/stokes_tables/archive/refs/heads/main.zip)
  - these contain:
    * `STOKES_model_definitions.xcm` - main script defining the STOKES models,
    * `load_null_data.xcm` - script loading the fake null data,
    * `stiso_model_example.xcm` - example script to view `stiso` model,
    * `stpol_model_example.xcm` - example script to view `stpol` model,
    * `stokes_model_example.xcm` - example script to view `stokes` model.
    * `README.md` - the documentation (this readme file)


Usage in XSPEC
==============

1. **Download the XSPEC scripts** ([stokes_tables-main.zip](https://github.com/jpodgorny/stokes_tables/archive/refs/heads/main.zip)) and uncompress them:  
  `unzip stokes_tables-main.zip`.

2. **Download the STOKES tables** 
  ([stokes_unpol_iso-v2.fits](https://owncloud.asu.cas.cz/index.php/s/lG7R3Ns5gDeDMkS), 
  [stokes_unpol-v2.fits](https://owncloud.asu.cas.cz/index.php/s/dtShnA2HYb6lvdR),
  [stokes_vrpol-v2.fits](https://owncloud.asu.cas.cz/index.php/s/06rK1zxDJMhb4CV),
  [stokes_45deg-v2.fits](https://owncloud.asu.cas.cz/index.php/s/BH6GMtbJUanNlB8)),
  or the reduced STOKES tables ([stokes_tables_reduced-v2.tar.gz](https://owncloud.asu.cas.cz/index.php/s/qOcBk05jPV4bQNR)). Note that the latter needs to be uncompressed with the command:  
  `tar -xzf stokes_tables_reduced-v2.tar.gz`.

3. **Download the package containing the fake null data and unit repsonses**
  ([fake_null_iqu_300ch.tar.gz](https://owncloud.asu.cas.cz/index.php/s/Flk6cwYLISmw0D5)) and uncompress it:  
  `tar -xzf fake_null_iqu_300ch.tar.gz`.

4. **Move the STOKES tables, null data and unit responses** to the directory with 
   the scripts (originally named `stokes_tables-main`). In case these will be put 
   into a different directory, you have to define the shell variables `STOKESDIR`
   and `NULLDATADIR` in `STOKES_model_definitions.xcm` and `load_null_data.xcm`
   scripts, respectively.

5. **Use the models** in XSPEC according to the example scripts provided, e.g.,  
   `@STOKES_model_definitions.xcm`

_Important notes:_

* When defining the models in XSPEC with `mdefine`, it is crucial to set 
the parameter values, including their limits, correctly during their first 
use with the `model` command. This is especially important when combining them with 
the mixing `polrot` model. **Incorrectly defined parameters may cause the models to 
produce undefined output, which can result in the `polrot` model crashing XSPEC!**

* Note that the XSPEC mixed model for the rotation of the Stokes parameters, 
`polrot`, must be used in combination with the above models, e.g. `polrot*stokes`,
to be able to fit for a specific orientation angle of the system.

* Note that the `mdefine` command does not work for polarisation models in XSPEC
versions 12.14.1b and earlier. To use the provided models, please, update your XSPEC
to a later version or try this [workaround](#workaround-for-xspec-versions-12141b-and-earlier).

Viewing the STOKES tables and models in XSPEC
=============================================

For viewing the STOKES tables and models using them inside XSPEC, we provide a fake
null data sets for all three Stokes parameters, i, q and u, binned in 300 channels, 
together with corresponding unit response matrices, rmf, arf and mrf, defined in 0.1 
to 100 keV in 300 channels, see 
[fake_null_iqu_300ch.tar.gz](https://owncloud.asu.cas.cz/index.php/s/Flk6cwYLISmw0D5). 
One can see the predicted polarisation properties in the following way:

1. **Download the package containing the fake null data and unit repsonses** - 
   [fake_null_iqu_300ch.tar.gz](https://owncloud.asu.cas.cz/index.php/s/Flk6cwYLISmw0D5)
   
2. **Uncompress the package**, e.g. by the command:

   `tar -xzf fake_null_iqu_300ch.tar.gz`

   This will uncompress the following files:
   
   - `fake_null_i_300ch.pha`
   - `fake_null_q_300ch.pha`
   - `fake_null_u_300ch.pha`
   - `fake_unit_response_0.1-100keV_300ch.rmf`
   - `fake_unit_response_0.1-100keV_300ch.arf`
   - `fake_unit_response_0.1-100keV_300ch.mrf`

3. **Load the fake null data into XSPEC**:

   `data 1 fake_null_i_300ch.pha`  
   `data 2 fake_null_q_300ch.pha`  
   `data 3 fake_null_u_300ch.pha`  
   `setplot energy`

   the fake unit responses will be automatically loaded.

4. **View the polarisation degree and angle** in XSPEC (after the model is defined with the `model` command):

   `plot polfrac`  
   `plot polangle`  

---

### Workaround for XSPEC versions 12.14.1b and earlier

The XSPEC `mdefine` command does not work for polarisation models in versions 12.14.1b and earlier. If one does not want to update XSPEC to later version and one has XSPEC installed from the source code, then the updated 
[`MdefExpression.cxx`](fix/MdefExpression.cxx?raw=1) file, kindly provided by Keith Arnaud, may fix the problem (tested with version 12.14.0h). 

Proceed in the following way to try this fix: 

* replace the original `MdefExpression.cxx` in `Xspec/src/XSFunctions/Utilities`
  with an updated [`MdefExpression.cxx`](fix/MdefExpression.cxx?raw=1) file,
  
* perform `touch MdefExpression.cxx` in `Xspec/src/XSFunctions/Utilities` to ensure the following step will recompile it,

* do `hmake` in `Xspec/src/XSFunctions` 
  (HEADAS has to be initialised for `hmake` to work),

* do `hmake install` in `Xspec/src/XSFunctions`.

Note that when defining the models in XSPEC with `mdefine`, it is crucial to set 
the parameter values, including their limits, correctly during their first 
use with the `model` command. This is especially important when combining them with 
the mixing `polrot` model. Incorrectly defined parameters may cause the models to 
produce undefined output, which can result in the `polrot` model crashing XSPEC.
