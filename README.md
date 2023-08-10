var query =
    from trd in dbContext.TrdReal
    join piutangSum in piutangSumQuery
        on new { Bilyet = trd.Bilyet, TanggalBayar = trd.TanggalBayar } equals
           new { Bilyet = piutangSum.Bilyet, TanggalBayar = piutangSum.TanggalBunga } into trdPiutangGroup
    from piutangSum in trdPiutangGroup.DefaultIfEmpty()
    where trd.TanggalBayar >= dtAwalValue && trd.TanggalBayar <= dtAkhirValue
          && (string.IsNullOrEmpty(MyFilter) || trd.MyFilterColumn == MyFilter)
    orderby trd.TanggalBayar descending
    select new
    {
        TanggalBayar = trd.TanggalBayar,
        Bilyet = trd.Bilyet,
        PiutangTotal = trd.BungaHitung - (piutangSum?.BiayaPiutang ?? 0) - (trd.Bunga ?? 0) - (trd.Biaya ?? 0),
        PiutangBayar = piutangSum?.PiutangBayar ?? 0,
        PiutangSisa = trd.BungaHitung - (piutangSum?.BiayaPiutang ?? 0) - (trd.Bunga ?? 0) - (trd.Biaya ?? 0) - (piutangSum?.PiutangBayar ?? 0),
        Piutang = piutangSum?.BiayaPiutang ?? 0,
        Lunas = trd.Lunas
    };

var result = query.ToList();
