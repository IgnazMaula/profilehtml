DateTime dtAwalValue = dtAwal.Value.Date;
DateTime dtAkhirValue = dtAkhir.Value.Date;

var piutangQuery = from piutangInner in dbContext.Piutang
                   where piutangInner.TanggalBunga >= dtAwalValue && piutangInner.TanggalBunga <= dtAkhirValue
                   group piutangInner by new { piutangInner.Bilyet, piutangInner.TanggalPiutang } into grp
                   select new
                   {
                       Bilyet = grp.Key.Bilyet,
                       TanggalPiutang = grp.Key.TanggalPiutang,
                       Baris = grp.Max(g => g.Line)
                   };

var piutangSumQuery = from piutang in dbContext.Piutang
                      join tes in piutangQuery
                      on new { piutang.Bilyet, piutang.TanggalPiutang, piutang.Line } equals new { tes.Bilyet, tes.TanggalPiutang, Line = tes.Baris }
                      where piutang.TanggalBunga >= dtAwalValue && piutang.TanggalBunga <= dtAkhirValue
                      group piutang by new { piutang.Bilyet, tes.TanggalPiutang } into grp
                      select new
                      {
                          Bilyet = grp.Key.Bilyet,
                          TanggalBunga = grp.Key.TanggalPiutang,
                          BiayaPiutang = grp.Sum(g => g.Biaya),
                          PiutangBayar = grp.Sum(g => g.PiutangBayar)
                      };

var query = from trd in dbContext.TrdReal
            join piutangSum in piutangSumQuery
            on new { trd.Bilyet, trd.TanggalBayar } equals new { piutangSum.Bilyet, piutangSum.TanggalBunga } into trdPiutangGroup
            from piutangSum in trdPiutangGroup.DefaultIfEmpty()
            where trd.TanggalBayar >= dtAwalValue && trd.TanggalBayar <= dtAkhirValue
                  && (string.IsNullOrEmpty(MyFilter) || trd.MyFilterColumn == MyFilter)
            orderby trd.TanggalBayar descending
            select new
            {
                TanggalBayar = trd.TanggalBayar,
                Bilyet = trd.Bilyet,
                PiutangTotal = trd.BungaHitung - (piutangSum.BiayaPiutang ?? 0) - (trd.Bunga ?? 0) - (trd.Biaya ?? 0),
                PiutangBayar = piutangSum.PiutangBayar ?? 0,
                PiutangSisa = trd.BungaHitung - (piutangSum.BiayaPiutang ?? 0) - (trd.Bunga ?? 0) - (trd.Biaya ?? 0) - (piutangSum.PiutangBayar ?? 0),
                Piutang = piutangSum.BiayaPiutang ?? 0,
                Lunas = trd.Lunas
            };

var result = query.ToList();
