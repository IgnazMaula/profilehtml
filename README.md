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
