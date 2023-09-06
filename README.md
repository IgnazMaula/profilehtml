     select distinct approvejt,trd_d_nos,trd_d_bank,trd_d_branch,trd_d_netamount, trd_D_DueDate from trd_d
        where approve = 1 and cair = 0 and ro = 0
        and year(TRD_D_DueDate)= '2023'
        and month(TRD_D_DueDate)= '05'
        and TRD_D_DP = 'DP Smart'
