// Step 1: Get the grouped Piutang data
var groupedPiutang = (
    from piutang in dbContext.Piutang
    where piutang.TanggalBunga >= dtAwalValue && piutang.TanggalBunga <= dtAkhirValue
    group piutang by new { piutang.Bilyet, piutang.TanggalPiutang } into g
    select new
    {
        Bilyet = g.Key.Bilyet,
        TanggalPiutang = g.Key.TanggalPiutang,
        BiayaPiutang = g.Sum(x => x.Biaya),
        PiutangBayar = g.Sum(x => x.PiutangBayar)
    }
).ToList(); // Execute the query and store the result in memory

// Step 2: Join TRD_REAL with the grouped Piutang data
var result = (
    from trd in dbContext.TrdReal
    join pg in groupedPiutang on new { trd.Bilyet, trd.TanggalBayar } equals new { pg.Bilyet, pg.TanggalPiutang }
    where trd.TanggalBayar >= dtAwalValue && trd.TanggalBayar <= dtAkhirValue
    select new
    {
        trd.TanggalBayar,
        trd.Bilyet,
        PiutangTotal = trd.BungaHitung - (pg.BiayaPiutang ?? 0) - (trd.Biaya ?? 0),
        PiutangBayar = pg.PiutangBayar ?? 0,
        PiutangSisa = trd.BungaHitung - (pg.BiayaPiutang ?? 0) - (trd.Biaya ?? 0) - (pg.PiutangBayar ?? 0) - (trd.BiayaPiutang ?? 0),
        trd.Piutang,
        trd.Lunas
    }
);

result = result.OrderByDescending(x => x.TanggalBayar);
