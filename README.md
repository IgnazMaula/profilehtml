var result = from tr in context.TRD_REAL
             join pi in (
                 from p in context.Piutang
                 join tes in (
                     from t in context.Piutang
                     where t.TanggalBunga >= startDate && t.TanggalBunga <= endDate
                     group t by new { t.Bilyet, t.TanggalPiutang } into g
                     select new { Bilyet = g.Key.Bilyet, TanggalPiutang = g.Key.TanggalPiutang, Baris = g.Max(t => t.Line) }
                 ) on new { p.Bilyet, p.TanggalPiutang, Line = p.Line } equals new { Bilyet = tes.Bilyet, TanggalPiutang = tes.TanggalPiutang, Line = tes.Baris }
                 where p.TanggalBunga >= startDate && p.TanggalBunga <= endDate
                 group p by new { p.Bilyet, p.TanggalBunga } into g
                 select new
                 {
                     Bilyet = g.Key.Bilyet,
                     TanggalBunga = g.Key.TanggalBunga,
                     BiayaPiutang = g.Sum(p => p.Biaya),
                     PiutangBayar = g.Sum(p => p.PiutangBayar)
                 }
             ) on new { tr.Bilyet, tr.TanggalBayar } equals new { pi.Bilyet, pi.TanggalBunga } into piGroup
             from pi in piGroup.DefaultIfEmpty()
             where tr.TanggalBayar >= startDate && tr.TanggalBayar <= endDate
             orderby tr.TanggalBayar descending
             select new
             {
                 tr.TanggalBayar,
                 tr.Bilyet,
                 PiutangTotal = tr.BungaHitung - (pi.BiayaPiutang ?? 0) - tr.Biaya - (pi.PiutangBayar ?? 0),
                 PiutangBayar = pi.PiutangBayar ?? 0,
                 PiutangSisa = tr.BungaHitung - (pi.BiayaPiutang ?? 0) - tr.Biaya - (pi.PiutangBayar ?? 0) - tr.BiayaPiutang,
                 Piutang = tr.Piutang,
                 tr.Lunas
             };

var queryResult = result.ToList();
