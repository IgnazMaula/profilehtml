select TRD_REAL.TanggalBayar, TRD_REAL.Bilyet as Bilyet,
BungaHitung-isnull(Bunga,0)-BiayaPiutang-isnull(Biaya,0) PiutangTotal,
isnull(PiutangBayar,0) PiutangBayar,
BungaHitung-isnull(Bunga,0)-isnull(Biaya,0)-isnull(PiutangBayar,0)-BiayaPiutang PiutangSisa, Piutang, Lunas
from trd_real left join
(select sum(Biaya) BiayaPiutang,Bilyet, TanggalBunga, sum(PiutangBayar) PiutangBayar from Piutang
inner join (select Bilyet bil,max(TanggalPiutang) tgl,max(line) baris from piutang 
where TanggalBunga between '2021-08-01 00:00:00.000' and '2023-08-1 00:00:00.000'  group by Bilyet, TanggalPiutang) TES on TES.baris = line and tes.tgl = TanggalPiutang and tes.bil = Bilyet 
where TanggalBunga between '2021-08-01 00:00:00.000' and '2023-08-1 00:00:00.000'  group by Bilyet, TanggalBunga)
Piutang on Piutang.Bilyet=TRD_REAL.Bilyet and Piutang.TanggalBunga=TRD_REAL.TanggalBayar
where TanggalBayar between '2021-08-01 00:00:00.000' and '2023-08-1 00:00:00.000'
order by TanggalBayar Desc

var startDate = new DateTime(2021, 8, 1);
var endDate = new DateTime(2023, 8, 1);

var startDate = new DateTime(2021, 8, 1);
var endDate = new DateTime(2023, 8, 1);

var result = (from trd in dbContext.TRD_REAL
              join piutang in
                  (from p in dbContext.Piutang
                   join tes in
                       (from pt in dbContext.Piutang
                        where pt.TanggalBunga >= startDate && pt.TanggalBunga <= endDate
                        group pt by new { pt.Bilyet, pt.TanggalPiutang } into g
                        select new
                        {
                            Bilyet = g.Key.Bilyet,
                            TanggalPiutang = g.Key.TanggalPiutang,
                            Baris = g.Max(x => x.Line)
                        })
                       on new { p.Line, p.TanggalPiutang, p.Bilyet } equals new { Line = tes.Baris, TanggalPiutang = tes.TanggalPiutang, Bilyet = tes.Bilyet }
                        where p.TanggalBunga >= startDate && p.TanggalBunga <= endDate
                        group p by new { p.Bilyet, p.TanggalBunga } into g
                        select new
                        {
                            Bilyet = g.Key.Bilyet,
                            TanggalBunga = g.Key.TanggalBunga,
                            BiayaPiutang = g.Sum(x => x.Biaya),
                            PiutangBayar = g.Sum(x => x.PiutangBayar)
                        })
                  on new { trd.Bilyet, trd.TanggalBayar } equals new { Bilyet = piutang.Bilyet, TanggalBayar = piutang.TanggalBunga } into piutangGroup
              from pg in piutangGroup.DefaultIfEmpty()
              where trd.TanggalBayar >= startDate && trd.TanggalBayar <= endDate
              orderby trd.TanggalBayar descending
              select new
              {
                  trd.TanggalBayar,
                  Bilyet = trd.Bilyet,
                  PiutangTotal = trd.BungaHitung - (pg.BiayaPiutang ?? 0) - (pg.Biaya ?? 0),
                  PiutangBayar = pg.PiutangBayar ?? 0,
                  PiutangSisa = trd.BungaHitung - (pg.Biaya ?? 0) - (pg.PiutangBayar ?? 0) - (pg.BiayaPiutang ?? 0),
                  Piutang = pg.BiayaPiutang ?? 0,
                  Lunas = trd.Lunas
              }).ToList();


