var result = from mshm in dbContext.MSHM
             join avg in dbContext.AVG on new { MSHM_C = mshm.Kode, DPCode = "DP Smart", Type = "SHM" } equals new { avg.MSHM_C, avg.DPCode, avg.Type } into avgJoin
             from avg in avgJoin.DefaultIfEmpty()
             join mfpihak in dbContext.MFPIHAK on mshm.MSHM_Desc equals mfpihak.MFPIHAK_Nama
             where mshm.MSHM_Active == 1
             orderby mshm.MSHM_C
             select new
             {
                 MSHM = mshm,
                 AVG = avg,
                 MFPIHAK = mfpihak
             };
