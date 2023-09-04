var result = (from trdReal in context.TRD_REAL
              where trdReal.Periode == "2007/10" && trdReal.DPCode == "DP Smart"
              join trdD in context.TRD_D on trdReal.NomorSuratBank equals trdD.NoSuratBank into trdDJoin
              from trdD in trdDJoin.DefaultIfEmpty()
              join mfbnk in context.MFBNK on new { trdReal.TRD_D_Bank, trdReal.TRD_D_Branch } equals new { mfbnk.Bank_ID, mfbnk.Branch_ID } into mfbnkJoin
              from mfbnk in mfbnkJoin.DefaultIfEmpty()
              orderby trdReal.bilyet, trdReal.nomorsuratbank
              select new
              {
                  bilyet = trdReal.bilyet,
                  nomorsuratbank = trdReal.nomorsuratbank
              }).Distinct();
