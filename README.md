var startDate = new DateTime(/* specify your start date */);
var endDate = new DateTime(/* specify your end date */);

// Retrieve TRD_REAL records within the specified date range
var trdRecords = context.TRD_REAL
    .Where(trd => trd.TanggalBayar >= startDate && trd.TanggalBayar <= endDate)
    .ToList();

// Retrieve Piutang records with associated data within the specified date range
var piutangRecords = (
    from pt in context.Piutang
    join tes in (
        from pt2 in context.Piutang
        where pt2.TanggalBunga >= startDate && pt2.TanggalBunga <= endDate
        group pt2 by new { pt2.Bilyet, pt2.TANGGALBUNGA } into g
        select new
        {
            g.Key.Bilyet,
            g.Key.TANGGALBUNGA,
            Baris = g.Max(x => x.line),
            Tgl = g.Max(x => x.TanggalPiutang)
        }
    ) on new { pt.line, pt.TanggalPiutang, pt.Bilyet } equals new { line = tes.Baris, TanggalPiutang = tes.Tgl, tes.Bilyet }
    where pt.TanggalBunga >= startDate && pt.TanggalBunga <= endDate
    group pt by new { pt.Bilyet, pt.TANGGALBUNGA } into g
    select new
    {
        g.Key.Bilyet,
        g.Key.TANGGALBUNGA,
        BiayaPiutang = g.Sum(x => x.Biaya),
        PiutangBayar = g.Sum(x => x.PiutangBayar)
    }
).ToList();

// Combine TRD_REAL and Piutang records into the ViewModel list
var result = (
    from trd in trdRecords
    join piutang in piutangRecords
    on new { trd.Bilyet, trd.TanggalBayar } equals new { piutang.Bilyet, piutang.TANGGALBUNGA } into piutangJoin
    from piutangData in piutangJoin.DefaultIfEmpty()
    let PiutangTotal = trd.BungaHitung - (piutangData?.BiayaPiutang ?? 0) - (trd.Biaya ?? 0)
    let PiutangBayar = piutangData?.PiutangBayar ?? 0
    let PiutangSisa = PiutangTotal - PiutangBayar - (trd.BiayaPiutang ?? 0)
    select new ViewModel
    {
        Trd = new Trd
        {
            TanggalBayar = trd.TanggalBayar,
            Bilyet = trd.Bilyet,
            BungaHitung = trd.BungaHitung,
            Biaya = trd.Biaya,
            Piutang = trd.Piutang,
            Lunas = trd.Lunas,
            PiutangTotal = PiutangTotal,
            PiutangBayar = PiutangBayar,
            PiutangSisa = PiutangSisa
        },
        Piutang = new Piutang
        {
            Bilyet = piutangData?.Bilyet,
            TANGGALBUNGA = piutangData?.TANGGALBUNGA,
            BiayaPiutang = piutangData?.BiayaPiutang ?? 0,
            PiutangBayar = piutangData?.PiutangBayar ?? 0
        }
    }
).OrderByDescending(r => r.Trd.TanggalBayar)
.ToList();
