select TRD_REAL.TanggalBayar, TRD_REAL.Bilyet as Bilyet,
BungaHitung-isnull(Bunga,0)-BiayaPiutang-isnull(Biaya,0) PiutangTotal,
isnull(PiutangBayar,0) PiutangBayar,
BungaHitung-isnull(Bunga,0)-isnull(Biaya,0)-isnull(PiutangBayar,0)-BiayaPiutang PiutangSisa, Piutang, Lunas
from trd_real left join
(select sum(Biaya) BiayaPiutang,Bilyet, TANGGALBUNGA, sum(PiutangBayar) PiutangBayar from Piutang
inner join (select Bilyet bil,max(TanggalPiutang) tgl,max(line) baris from piutang 
where TanggalBunga between 'startDate' and 'endDate'  group by Bilyet, TanggalPiutang) TES on TES.baris = line and tes.tgl = TanggalPiutang and tes.bil = Bilyet 
where TanggalBunga between 'startDate' and 'endDate'  group by Bilyet, TANGGALBUNGA)
Piutang on Piutang.Bilyet=TRD_REAL.Bilyet and Piutang.TanggalBunga=TRD_REAL.TanggalBayar
where TanggalBayar between 'startDate' and 'endDate'
order by TanggalBayar Desc
