// Subquery to calculate nominalJual
var subqueryY = from b in context.TRD_OH_SBP
                join m in context.MOBL on b.TRD_OH_Kode equals m.MOBL_Code
                where m.MOBL_MaturityDate.Year == 2023 && m.MOBL_MaturityDate.Month == 7
                select new
                {
                    b.ApproveJT,
                    b.TRD_OH_Kode,
                    b.Nominal,
                    m.MOBL_IssuedDate,
                    m.MOBL_MaturityDate,
                    nominalJual = (from s in context.TRD_OH_SBP_Sell
                                   where s.noBeli == b.TRD_OH_NO
                                   select (decimal?)s.Nominal).Sum() ?? 0
                };

// Execute the subquery and project it to a list
var subqueryYList = subqueryY.ToList();

// Perform the grouping and aggregation in memory
var result = subqueryYList.GroupBy(x => new
               {
                   x.ApproveJT,
                   x.TRD_OH_Kode,
                   x.MOBL_IssuedDate,
                   x.MOBL_MaturityDate
               })
               .Select(g => new
               {
                   g.Key.ApproveJT,
                   g.Key.TRD_OH_Kode,
                   Nominal = g.Sum(x => x.Nominal + x.nominalJual),
                   g.Key.MOBL_IssuedDate,
                   g.Key.MOBL_MaturityDate
               })
               .OrderBy(x => x.MOBL_MaturityDate)
               .ToList();
