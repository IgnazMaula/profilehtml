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
                      from piutangResult in piutangJoin.DefaultIfEmpty()
                      where piutangResult != null && piutangResult.TanggalPiutang >= dtAwal.Value && piutangResult.TanggalPiutang <= dtAkhir.Value
                      group piutangResult by new { piutangResult.Bilyet, piutangResult.TanggalPiutang } into g
                      select new
                      {
                          Bilyet = g.Key.Bilyet,
                          TanggalBunga = g.Key.TanggalPiutang,
                          BiayaPiutang = g.Sum(p => p.Biaya),
                          PiutangBayar = g.Sum(p => p.PiutangBayar)
                      };

var query = from trd in YourDbContext.TrdReal
            join piutang in piutangSubquery on new { trd.Bilyet, trd.TanggalBayar } equals new { piutang.Bilyet, piutang.TanggalBunga } into joinedPiutang
            from piutangResult in joinedPiutang.DefaultIfEmpty()
            where trd.TanggalBayar >= dtAwal.Value && trd.TanggalBayar <= dtAkhir.Value && MyFilterConditions
            orderby trd.TanggalBayar descending
            select new
            {
                trd.TanggalBayar,
                Bilyet = trd.Bilyet,
                PiutangTotal = trd.BungaHitung - (piutangResult?.BiayaPiutang ?? 0) - (piutangResult?.PiutangBayar ?? 0),
                PiutangBayar = piutangResult?.PiutangBayar ?? 0,
                PiutangSisa = trd.BungaHitung - (piutangResult?.Biaya ?? 0) - (piutangResult?.PiutangBayar ?? 0) - (piutangResult?.BiayaPiutang ?? 0),
                Piutang = piutangResult?.BiayaPiutang,
                Lunas = trd.Lunas
            };

var result = query.ToList();
