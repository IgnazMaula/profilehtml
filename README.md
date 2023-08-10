using System;
using System.Linq;

// Assuming you have an Entity Framework DbContext named 'YourDbContext'

var piutangSubquery = from piutang in YourDbContext.Piutang
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
                      into piutangJoin
                      from piutangResult in piutangJoin.DefaultIfEmpty()
                      where piutangResult.TanggalBunga >= dtAwal.Value && piutangResult.TanggalBunga <= dtAkhir.Value
                      group piutangResult by new { piutangResult.Bilyet, piutangResult.TanggalBunga } into g
                      select new
                      {
                          Bilyet = g.Key.Bilyet,
                          TanggalBunga = g.Key.TanggalBunga,
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
