var total = dbContext.TrdSaham
    .Where(s => s.Kodesaham == tKode && s.Tanggalsaham <= tDate && s.DP == tdp)
    .GroupBy(s => s.Kodesaham)
    .Select(g => new
    {
        Total = g.Sum(s => s.Totalsaham)
    })
    .FirstOrDefault()?.Total;
