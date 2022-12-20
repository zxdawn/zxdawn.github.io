---
title: Map MEIC to MOZCART
date: 2020-03-10
tags: ["WRF-Chem"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-03/emission.jpg"
summary: "How to map MEIC emission inventories to MOZART in WRF-Chem"
---

The species associated with specific chemical mechanism is list in `Registry/registry.chem` after `chem_opt==` and `emi_opt=`.

I will focus on the MOZCART mechanism (8):

```
# KPP mechanism from mozart + gocart
package   mozcart_kpp       chem_opt==112                  -          chem:o3,o1d_cb4,o,no,no2,no3,n2o5,hno3,hno4,so2,ho,ho2,h2o2,sulf,co,hcho,ch3ooh,ch3o2,ch4,h2,eo2,ch3cooh,c2h4,n2o,ch3oh,aco3,acet,mgly,paa,gly,c3h6ooh,pan,mpan,macr,mvk,c3h6,etooh,prooh,acetp,xooh,onitr,isooh,acetol,glyald,mek,eto2,open,alkooh,mekooh,tolooh,terpooh,ald,mco3,c2h5oh,eo,c2h6,c3h8,pro2,po2,aceto2,bigene,bigalk,eneo2,alko2,isopr,iso2,mvko2,mvkooh,hydrald,xo2,c10h16,terpo2,tol,cres,to2,xoh,onit,isopn,dms,nh3,meko2,p25,bc1,bc2,oc1,oc2,dust_1,dust_2,dust_3,dust_4,dust_5,seas_1,seas_2,seas_3,seas_4,p10

.....

package   mozcem          emiss_opt==8                   -             emis_ant:e_co,e_no,e_no2,e_bigalk,e_bigene,e_c2h4,e_c2h5oh,e_c2h6,e_c3h6,e_c3h8,e_ch2o,e_ch3cho,e_ch3coch3,e_ch3oh,e_mek,e_so2,e_toluene,e_nh3,e_isop,e_c10h16,e_pm_10,e_pm_25,e_bc,e_oc,e_sulf
```

Here's the table of approximate matching of MOZCART species (listed after `Anthropogenic emissions` in `Registry/registry.chem`) to suffixes of [MEIC](http://meicmodel.org/cloud/) files with different mechanisms.



<style>
table {
    width:100%;
}
</style>

| WRF-Chem (MOZCART) |            CB05             |    SAPRC99     |    RADM2     |
| :----------------: | :-------------------------: | :------------: | :----------: |
|      E_C10H16      |          **TERP**           |                |              |
|      E_C2H5OH      |          **ETOH**           |                |              |
|      E_CH3OH       |            MEOH             |    **MEOH**    |              |
|       E_C3H6       |          OLE + PAR          |    **OLE1**    |              |
|      E_BIGENE      | OLE + 2\*PAR, IOLE + 2\*PAR |    **OLE2**    |  OLT + OLI   |
|       E_MEK        |                             |    MEK+PRD2    |   **KET**    |
|       E_NH3        |             NH3             |      NH3       |   **NH3**    |
|      E_PM_25       |            PM2.5            |     PM2.5      |  **PM2.5**   |
|      E_PM_10       |          PMcoarse           |    PMcoarse    | **PMcoarse** |
|      E_BIGALK      |           5\*PAR            | ALK3+ALK4+ALK5 |   **HC5**    |
|       E_C2H4       |             ETH             |      ETHE      |   **OL2**    |
|       E_C2H6       |            ETHA             |      ALK1      |   **ETH**    |
|       E_C3H8       |       1.12\*10-2 \*CO       |      ALK2      |   **HC3**    |
|       E_CH2O       |            FORM             |      HCHO      |   **HCHO**   |
|      E_CH3CHO      |          ALD2+ALDX          |      CCHO      |   **ALD**    |
|     E_CH3COCH3     |       1.18\*10-2 \*CO       |      ACET      |   **KET**    |
|     E_TOLUENE      |             TOL             |      ARO1      |   **TOL**    |
|       E_ISOP       |            ISOP             |      ISOP      |   **ISO**    |
|       E_SO2        |             SO2             |      SO2       |   **SO2**    |
|        E_NO        |          0.9\*NOx           |    0.9\*NOx    | **0.9\*NOx** |
|       E_NO2        |          0.1\*NOx           |    0.1\*NOx    | **0.1\*NOx** |
|        E_CO        |             CO              |       CO       |    **CO**    |

In case we wanna switch to mechanism similar with MOZCART, I listed some other species below:

<style>
table {
    width:100%;
}
</style>

| WRF-Chem  |        CB05        | SAPRC99  | SAPRC07  |  RADM2   |
| :-------: | :----------------: | :------: | :------: | :------: |
|  E_C2H2   | **5.87\*10-3\*CO** |          |          |          |
|  E_MACR   |                    | **MACR** |          |          |
| E_BENZENE |                    |          | **BENZ** |          |
|   E_HC3   |                    |          |          | **HC3**  |
|   E_HC5   |                    |          |          | **HC5**  |
|   E_HC8   |                    |          |          | **HC8**  |
|   E_ISO   |                    |          |          | **ISO**  |
|   E_OL2   |                    |          |          | **OL2**  |
|   E_OLT   |                    |          |          | **OLT**  |
|   E_OLI   |                    |          |          | **OLI**  |
|   E_TOL   |                    |          |          | **TOL**  |
|   E_CSL   |                    |          |          | **CSL**  |
|   E_ALD   |                    |          |          | **ALD**  |
|   E_KET   |                    |          |          | **KET**  |
|  E_ORA2   |                    |          |          | **ORA2** |
| E_CH3COOH |                    |          |          | **ORA2** |
|   E_MVK   |   2.40\*10-4\*CO   | **MVK**  |          |          |
|   E_XYL   |        XYL         |          |          | **XYL**  |
|   E_ETH   |        ETH         |          |          | **ETH**  |
|  E_HCHO   |                    |   HCHO   |          | **HCHO** |
| E_XYLENE  |        XYL         |   ARO2   |          | **XYL**  |
|   E_CO2   |        CO2         |   CO2    |          | **CO2**  |



## Reference

1. http://www.geosci-model-dev.net/3/43/2010/
2. [MOZART_MOSAIC User's Guide](https://www2.acom.ucar.edu/sites/default/files/wrf-chem/MOZART_MOSAIC_V3.6.readme_dec2016.pdf)
3. [Information for mapping CESM2 (CAM-chem or WACCM) gas and aerosol species to WRF-Chem species for various mechanisms](https://www2.acom.ucar.edu/sites/default/files/wrf-chem/CESM-WRFchem_aerosols_20190822.pdf)
4.  [MOZCART User's Guide](http://www.acom.ucar.edu/wrf-chem/MOZCART_UsersGuide.pdf)
5. [About chemical speciation processing of EPA-2014v2](https://groups.google.com/a/ucar.edu/d/msg/wrf-chem-anthro_emiss/aJlGhAtJyW4/ly76j99XDQAJ)
6. Photo by [veeterzy](https://unsplash.com/@veeterzy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## Version control

| Version | Action               | Time       |
| ------- | -------------------- | ---------- |
| 1.0     | Init                 | 2020-03-10 |
| 1.1     | Update Species Table | 2020-03-13 |