# using System;
using System.Linq;

// Assuming you have an Entity Framework DbContext named 'YourDbContext'

var query = from trd in YourDbContext.TrdReal
            join piutangSubquery in (
                from piutang in YourDbContext.Piutang
                join tesSubquery in (
                    from p in YourDbContext.Piutang
                    where p.TanggalBunga >= dtAwal.Value && p.TanggalBunga <= dtAkhir.Value
                    group p by new { p.Bilyet, p.TanggalPiutang } into g
                    select new
                    {
                        Bilyet = g.Key.Bilyet,
                        TanggalPiutang = g.Key.TanggalPiutang,
                        MaxLine = g.Max(p => p.Line)
                    }
                ) on new { piutang.Bilyet, piutang.TanggalPiutang, piutang.Line } equals new { tesSubquery.Bilyet, tesSubquery.TanggalPiutang, Line = tesSubquery.MaxLine }
                where piutang.TanggalBunga >= dtAwal.Value && piutang.TanggalBunga <= dtAkhir.Value
                group piutang by new { piutang.Bilyet, piutang.TanggalBunga } into g
                select new
                {
                    Bilyet = g.Key.Bilyet,
                    TanggalBunga = g.Key.TanggalBunga,
                    BiayaPiutang = g.Sum(p => p.Biaya),
                    PiutangBayar = g.Sum(p => p.PiutangBayar)
                }
            ) on new { trd.Bilyet, trd.TanggalBayar } equals new { piutangSubquery.Bilyet, piutangSubquery.TanggalBunga } into piutangJoin
            from piutang in piutangJoin.DefaultIfEmpty()
            where trd.TanggalBayar >= dtAwal.Value && trd.TanggalBayar <= dtAkhir.Value && MyFilterConditions
            orderby trd.TanggalBayar descending
            select new
            {
                trd.TanggalBayar,
                Bilyet = trd.Bilyet,
                PiutangTotal = trd.BungaHitung - (piutang.BiayaPiutang ?? 0) - (piutangSubquery.PiutangBayar ?? 0),
                PiutangBayar = piutangSubquery.PiutangBayar ?? 0,
                PiutangSisa = trd.BungaHitung - (piutang.Biaya ?? 0) - (piutangSubquery.PiutangBayar ?? 0) - (piutangSubquery.BiayaPiutang ?? 0),
                Piutang = piutangSubquery.BiayaPiutang,
                Lunas = trd.Lunas
            };

var result = query.ToList();
