var query = from trdReal in context.TRD_REAL
            join trdD in context.TRD_D on trdReal.NomorSuratBank equals trdD.NoSuratBank
            join mfbnk in context.MFBNK on new { trdReal.TRD_D_Bank, trdReal.TRD_D_Branch } equals new { mfbnk.Bank_ID, mfbnk.Branch_ID }
            where trdReal.DPCode == "DP Smart"
            orderby trdReal.bilyet, trdReal.nomorsuratbank ascending
            select trdReal;
