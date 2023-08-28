var result = from mshm in context.MSHM
             join avg in context.AVG on new { MSHM_C = mshm.Kode } equals new { avg.MSHM_C } into avgGroup
             from avgData in avgGroup.Where(a => a.DPCode == "DP Smart" && a.Type == "SHM").DefaultIfEmpty()
             join mfpihak in context.MFPIHAK on mshm.MSHM_Desc equals mfpihak.MFPIHAK_Nama
             where mshm.MSHM_Active == 1
             orderby mshm.MSHM_C
             select new
             {
                 MSHM = mshm,
                 AVG = avgData,
                 MFPIHAK = mfpihak
             };
