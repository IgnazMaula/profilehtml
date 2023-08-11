var result = from tr in context.TRD_REAL
             where tr.TanggalBayar >= startDate && tr.TanggalBayar <= endDate
             let piGroup = (
                 from p in context.Piutang
                 where p.TanggalBunga >= startDate && p.TanggalBunga <= endDate
                 group p by new { p.Bilyet, p.TanggalBunga } into g
                 select new
                 {
                     g.Key.Bilyet,
                     g.Key.TanggalBunga,
                     BiayaPiutang = g.Sum(p => p.Biaya),
                     PiutangBayar = g.Sum(p => p.PiutangBayar)
                 }
             )
             from pi in piGroup
                 .Where(p => p.Bilyet == tr.Bilyet && p.TanggalBunga == tr.TanggalBayar)
                 .DefaultIfEmpty()
             orderby tr.TanggalBayar descending
             select new
             {
                 tr.TanggalBayar,
                 tr.Bilyet,
                 PiutangTotal = tr.BungaHitung - (pi.BiayaPiutang ?? 0) - tr.Biaya - (pi.PiutangBayar ?? 0),
                 PiutangBayar = pi.PiutangBayar ?? 0,
                 PiutangSisa = tr.BungaHitung - (pi.BiayaPiutang ?? 0) - tr.Biaya - (pi.PiutangBayar ?? 0) - tr.BiayaPiutang,
                 tr.Piutang,
                 tr.Lunas
             };

var queryResult = result.ToList();
