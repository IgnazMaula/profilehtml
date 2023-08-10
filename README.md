DateTime dtAwalValue = dtAwal.Value;
DateTime dtAkhirValue = dtAkhir.Value;

var piutangGrouped = (
    from piutang in dbContext.Piutang
    join innerPiutang in (
        from ip in dbContext.Piutang
        where ip.TanggalBunga >= dtAwalValue && ip.TanggalBunga <= dtAkhirValue
        group ip by new { ip.Bilyet } into g
        select new { Bilyet = g.Key.Bilyet, TanggalPiutang = g.Max(x => x.TanggalPiutang), Line = g.Max(x => x.Line) }
    ) on new { piutang.Bilyet, piutang.TanggalPiutang, piutang.Line } equals new { innerPiutang.Bilyet, innerPiutang.TanggalPiutang, innerPiutang.Line }
    where piutang.TanggalBunga >= dtAwalValue && piutang.TanggalBunga <= dtAkhirValue
    group piutang by new { piutang.Bilyet, piutang.TanggalBunga } into g
    select new
    {
        Bilyet = g.Key.Bilyet,
        TanggalBunga = g.Key.TanggalBunga,
        BiayaPiutang = g.Sum(x => x.Biaya),
        PiutangBayar = g.Sum(x => x.PiutangBayar)
    }
);

var result = (
    from trd in dbContext.TrdReal
    join pg in piutangGrouped on new { trd.Bilyet, trd.TanggalBayar } equals new { pg.Bilyet, pg.TanggalBunga }
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
