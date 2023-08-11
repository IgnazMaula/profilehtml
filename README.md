string sql = @"
    select TRD_REAL.TanggalBayar, TRD_REAL.Bilyet as Bilyet, BungaHitung-isnull(Bunga,0)-BiayaPiutang-isnull(Biaya,0) PiutangTotal,
    isnull(PiutangBayar,0) PiutangBayar,
    BungaHitung-isnull(Bunga,0)-isnull(Biaya,0)-isnull(PiutangBayar,0)-BiayaPiutang PiutangSisa, Piutang, Lunas
    from trd_real
    left join
    (select sum(Biaya) BiayaPiutang, Bilyet, TANGGALBUNGA, sum(PiutangBayar) PiutangBayar from Piutang
    inner join (select Bilyet bil, max(TanggalPiutang) tgl, max(line) baris from piutang where TanggalBunga between @dtAwal and @dtAkhir group by Bilyet, TanggalPiutang) TES
    on TES.baris = line and tes.tgl = TanggalPiutang and tes.bil = Bilyet
    where TanggalBunga between @dtAwal and @dtAkhir group by Bilyet, TANGGALBUNGA)
    Piutang on Piutang.Bilyet = TRD_REAL.Bilyet and Piutang.TanggalBunga = TRD_REAL.TanggalBayar
    where TanggalBayar between @dtAwal and @dtAkhir
    order by TanggalBayar Desc;
";
In the provided SQL query string, I replaced the dynamic date values dtAwal and dtAkhir with parameter placeholders @dtAwal and @dtAkhir. This will allow you to pass in the actual date values as parameters when executing the query.

When executing this query using Entity Framework's Database.SqlQuery method (as shown in previous responses), you'll provide the parameters like this:

csharp
Copy code
object[] parameters = { dtAwal, dtAkhir };
_dbContext.Database.SqlQuery<YourResultType>(sql, parameters).ToList();
Please replace YourResultType with the appropriate class to match the query result columns and adjust the code to fit your application's structure.





