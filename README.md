select * from MSHM left join AVG on MSHM_C=Kode and DPCode='DP Smart' and Type='SHM' left join MFPIHAK on MFPIHAK_Nama=MSHM_Desc where MSHM_Active=1 ORDER BY MSHM_C
