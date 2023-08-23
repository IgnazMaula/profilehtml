var query = from trd in dbContext.TRD_REAL_HTM
            join piutang in dbContext.Piutang on new { Kode = trd.Kode, TanggalBunga = trd.TanggalTerima, Type = "HTM" } equals new { piutang.Kode, piutang.TanggalBunga, piutang.Type } into piutangJoin
            from piutang in piutangJoin.DefaultIfEmpty()
            let pValue = piutang != null ? piutang.Piutang : (trd.BungaHitung - trd.Bunga - trd.Biaya)
            where trd.TanggalTerima >= startDate && trd.TanggalTerima < endDate
            orderby trd.TanggalTerima descending
            select new
            {
                s = piutang != null ? piutang.Piutang : 0,
                r = trd.Kode,
                q = piutang != null ? piutang.Line : null,
                p = pValue,
                trd.Bank,
                trd.NoRekening,
                trd
            };
Please note that in the LINQ query, I assumed that you have a DataContext named dbContext that is used to interact with the database, and that the TRD_REAL_HTM and Piutang tables are represented as objects in the context. You might need to adjust the actual property


