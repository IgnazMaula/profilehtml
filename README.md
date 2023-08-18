select isnull(Piutang.Piutang,0) as s, 
TRD_REAL_HTM.Kode as r, Piutang.Line as q, 
isnull(Piutang.Piutang, BungaHitung-Bunga-Piutang.Biaya) as p, Bank, NoRekening,* 
from trd_real_HTM left join Piutang on Type='HTM' and Piutang.Bilyet=TRD_REAL_HTM.Kode and Piutang.TanggalBunga=TRD_REAL_HTM.TanggalTerima 
where TanggalTerima between '2021-08-01 00:00:00.000' and '2023-08-01 00:00:00.000'
order by TanggalTerima desc
