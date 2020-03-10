---
title: Map MEIC to MOZART
date: 2020-03-10
tags: ["WRF-Chem"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-03/emission.jpg"
summary: "How to map MEIC emission inventories to MOZART in WRF-Chem"
---

Approximate matching of MOZART species to suffixes of [MEIC](http://meicmodel.org/cloud/) files with different mechanisms.

<style>
table {
    width:100%;
}
</style>

|  WRF-Chem  |            CB05             |    SAPRC-99    |    RADM2     |
| :--------: | :-------------------------: | :------------: | :----------: |
|  E_C10H16  |          **TERP**           |                |              |
|  E_C2H5OH  |          **ETOH**           |                |              |
|   E_C2H2   |     **5.87\*10-3\*CO**      |                |              |
|   E_MACR   |                             |    **MACR**    |              |
| E_BENZENE  |                             |    **BENZ**    |              |
|   E_ISO    |                             |                |   **ISO**    |
|   E_HC3    |                             |                |   **HC3**    |
|   E_HC5    |                             |                |   **HC5**    |
|   E_HC8    |                             |                |   **HC8**    |
|   E_OL2    |                             |                |   **OL2**    |
|   E_OLT    |                             |                |   **OLT**    |
|   E_OLI    |                             |                |   **OLI**    |
|   E_TOL    |                             |                |   **TOL**    |
|   E_CSL    |                             |                |   **CSL**    |
|   E_ALD    |                             |                |   **ALD**    |
|   E_KET    |                             |                |   **KET**    |
|   E_ORA2   |                             |                |   **ORA2**   |
| E_CH3COOH  |                             |                |   **ORA2**   |
|  E_CH3OH   |            MEOH             |    **MEOH**    |              |
|   E_C3H6   |          OLE + PAR          |    **OLE1**    |              |
|   E_MVK    |       2.40\*10-4\*CO        |    **MVK**     |              |
|   E_XYL    |             XYL             |                |   **XYL**    |
|   E_ETH    |             ETH             |                |   **ETH**    |
|   E_HCHO   |                             |      HCHO      |   **HCHO**   |
|   E_MEK    |                             |    MEK+PRD2    |   **KET**    |
|   E_NH3    |             NH3             |      NH3       |   **NH3**    |
|  E_PM_25   |            PM2.5            |     PM2.5      |  **PM2.5**   |
|  E_PM_10   |          PMcoarse           |    PMcoarse    | **PMcoarse** |
|  E_BIGALK  |           5\*PAR            | ALK3+ALK4+ALK5 |   **HC5**    |
|  E_BIGENE  | OLE + 2\*PAR, IOLE + 2\*PAR |    **OLE2**    |  OLT + OLI   |
|   E_C2H4   |             ETH             |      ETHE      |   **OL2**    |
|   E_C2H6   |            ETHA             |      ALK1      |   **ETH**    |
|   E_C3H8   |       1.12\*10-2 \*CO       |      ALK2      |   **HC3**    |
|   E_CH2O   |            FORM             |      HCHO      |   **HCHO**   |
|  E_CH3CHO  |          ALD2+ALDX          |      CCHO      |   **ALD**    |
| E_CH3COCH3 |       1.18\*10-2 \*CO       |      ACET      |   **KET**    |
| E_TOLUENE  |             TOL             |      ARO1      |   **TOL**    |
|  E_XYLENE  |             XYL             |      ARO2      |   **XYL**    |
|   E_ISOP   |            ISOP             |      ISOP      |   **ISO**    |
|   E_SO2    |             SO2             |      SO2       |   **SO2**    |
|    E_NO    |          0.9\*NOx           |    0.9\*NOx    | **0.9\*NOx** |
|   E_NO2    |          0.1\*NOx           |    0.1\*NOx    | **0.1\*NOx** |
|    E_CO    |             CO              |       CO       |    **CO**    |
|   E_CO2    |             CO2             |      CO2       |   **CO2**    |

## Reference

1. http://www.geosci-model-dev.net/3/43/2010/
2. [MOZART_MOSAIC User's Guide](https://www2.acom.ucar.edu/sites/default/files/wrf-chem/MOZART_MOSAIC_V3.6.readme_dec2016.pdf)
3. [Information for mapping CESM2 (CAM-chem or WACCM) gas and aerosol species to WRF-Chem species for various mechanisms](https://www2.acom.ucar.edu/sites/default/files/wrf-chem/CESM-WRFchem_aerosols_20190822.pdf)
4.  [MOZCART User's Guide](http://www.acom.ucar.edu/wrf-chem/MOZCART_UsersGuide.pdf)
5. Photo by [veeterzy](https://unsplash.com/@veeterzy?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## Version control

| Version | Action | Time       |
| ------- | ------ | ---------- |
| 1.0     | Init   | 2020-03-10 |