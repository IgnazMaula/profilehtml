var query = from trd in _dbContext.TrdReals
                    join piutang in _dbContext.Piutangs
                        on new { trd.Bilyet, trd.TanggalBayar } equals new { piutang.Bilyet, piutang.TanggalBunga }
                    where trd.TanggalBayar >= dtAwal && trd.TanggalBayar <= dtAkhir
                          && (string.IsNullOrEmpty(MyFilter) || trd.SomeProperty == MyFilter)
                    select new
                    {
                        trd.TanggalBayar,
                        trd.Bilyet,
                        PiutangTotal = trd.BungaHitung - (piutang.Bunga ?? 0) - (piutang.Biaya ?? 0),
                        PiutangBayar = piutang.PiutangBayar ?? 0,
                        PiutangSisa = trd.BungaHitung - (piutang.Bunga ?? 0) - (piutang.Biaya ?? 0) - (piutang.PiutangBayar ?? 0),
                        piutang.Piutang,
                        piutang.Lunas
                    };

        return query.OrderByDescending(item => item.TanggalBayar);
