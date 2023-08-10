DateTime dtAwalValue = dtAwal.Value;
DateTime dtAkhirValue = dtAkhir.Value;

var result = from trd in dbContext.TrdReal
             join piutangGrouped in (
                 from piutang in dbContext.Piutang
                 join tes in (
                     from innerPiutang in dbContext.Piutang
                     where innerPiutang.TanggalBunga >= dtAwalValue && innerPiutang.TanggalBunga <= dtAkhirValue
                     group innerPiutang by new { innerPiutang.Bilyet, innerPiutang.TanggalPiutang } into g
                     select new { Bilyet = g.Key.Bilyet, TanggalPiutang = g.Key.TanggalPiutang, Line = g.Max(x => x.Line) }
                 ) on new { piutang.Bilyet, piutang.TanggalPiutang, Line = piutang.Line } equals new { tes.Bilyet, tes.TanggalPiutang, Line = tes.Line }
                 where piutang.TanggalBunga >= dtAwalValue && piutang.TanggalBunga <= dtAkhirValue
                 group piutang by new { piutang.Bilyet, piutang.TanggalBunga } into g
                 select new
                 {
                     Bilyet = g.Key.Bilyet,
                     TanggalBunga = g.Key.TanggalBunga,
                     BiayaPiutang = g.Sum(x => x.Biaya),
                     PiutangBayar = g.Sum(x => x.PiutangBayar)
                 }
             ) on new { trd.Bilyet, trd.TanggalBayar } equals new { piutangGrouped.Bilyet, piutangGrouped.TanggalBunga }
             where trd.TanggalBayar >= dtAwalValue && trd.TanggalBayar <= dtAkhirValue
             select new
             {
                 trd.TanggalBayar,
                 trd.Bilyet,
                 PiutangTotal = trd.BungaHitung - (piutangGrouped.BiayaPiutang ?? 0) - (trd.Biaya ?? 0),
                 PiutangBayar = piutangGrouped.PiutangBayar ?? 0,
                 PiutangSisa = trd.BungaHitung - (piutangGrouped.BiayaPiutang ?? 0) - (trd.Biaya ?? 0) - (piutangGrouped.PiutangBayar ?? 0) - (trd.BiayaPiutang ?? 0),
                 trd.Piutang,
                 trd.Lunas
             };

result = result.OrderByDescending(x => x.TanggalBayar);
