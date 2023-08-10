using System;
using System.Linq;

// Assuming you have an Entity Framework DbContext named 'YourDbContext'

var piutangSubquery = from p1 in YourDbContext.Piutang
                      join tesSubquery in (
                          from p2 in YourDbContext.Piutang
                          where p2.TanggalBunga >= dtAwal.Value && p2.TanggalBunga <= dtAkhir.Value
                          group p2 by new { p2.Bilyet, p2.TanggalPiutang } into g
                          select new
                          {
                              Bilyet = g.Key.Bilyet,
                              TanggalPiutang = g.Key.TanggalPiutang,
                              MaxLine = g.Max(p2 => p2.Line)
                          }
                      ) on new { p1.Bilyet, p1.TanggalPiutang, p1.Line } equals new { tesSubquery.Bilyet, tesSubquery.TanggalPiutang, Line = tesSubquery.MaxLine }
                      into piutangJoin
                      from piutang in piutangJoin.DefaultIfEmpty()
                      where p1.TanggalBunga >= dtAwal.Value && p1.TanggalBunga <= dtAkhir.Value
                      group p1 by new { p1.Bilyet, p1.TanggalBunga } into g
                      select new
                      {
                          Bilyet = g.Key.Bilyet,
                          TanggalBunga = g.Key.TanggalBunga,
                          BiayaPiutang = g.Sum(p1 => p1.Biaya),
                          PiutangBayar = g.Sum(p1 => p1.PiutangBayar)
                      };

var query = from trd in YourDbContext.TrdReal
            join piutangResult in piutangSubquery on new { trd.Bilyet, trd.TanggalBayar } equals new { piutangResult.Bilyet, piutangResult.TanggalBunga } into joinedPiutang
            from piutang in joinedPiutang.DefaultIfEmpty()
            where trd.TanggalBayar >= dtAwal.Value && trd.TanggalBayar <= dtAkhir.Value && MyFilterConditions
            orderby trd.TanggalBayar descending
            select new
            {
                trd.TanggalBayar,
                Bilyet = trd.Bilyet,
                PiutangTotal = trd.BungaHitung - (piutang?.BiayaPiutang ?? 0) - (piutang?.PiutangBayar ?? 0),
                PiutangBayar = piutang?.PiutangBayar ?? 0,
                PiutangSisa = trd.BungaHitung - (piutang?.Biaya ?? 0) - (piutang?.PiutangBayar ?? 0) - (piutang?.BiayaPiutang ?? 0),
                Piutang = piutang?.BiayaPiutang,
                Lunas = trd.Lunas
            };

var result = query.ToList();
