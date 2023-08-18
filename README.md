var result = from trd in dbContext.TRD_REAL_HTM
             join piutang in dbContext.Piutang on new { Kode = trd.Kode, TanggalBunga = trd.TanggalTerima }
                                              equals new { Kode = piutang.Bilyet, TanggalBunga = piutang.TanggalBunga }
                                              into piutangJoin
             from piutang in piutangJoin.DefaultIfEmpty()
             where trd.TanggalTerima >= new DateTime(2021, 8, 1) && trd.TanggalTerima < new DateTime(2023, 8, 1)
             orderby trd.TanggalTerima descending
             select new
             {
                 s = (piutang.Piutang ?? 0),
                 r = trd.Kode,
                 q = piutang.Line,
                 p = (piutang.Piutang ?? trd.BungaHitung - trd.Bunga - piutang.Biaya),
                 trd.Bank,
                 trd.NoRekening,
                 trd.*
             };
