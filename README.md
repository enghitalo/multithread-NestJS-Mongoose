# multithread-NestJS-Mongoose

      const cpuCount = require("os").cpus().length; // 8     
      var campanyList: any[] = [];
      var today: any = new Date();
      var firstDay: any = new Date("2018-03-14T20:13:08.667Z"); //2018-03-14T20:13:08.667Z
      const folga = (today - firstDay) / cpuCount;

       CompanyModel.find({})
        .limit(1)
        .lean()
        .then((value) => {

          firstDay = value[0].createdAt;

          for (let c = 1; c <= cpuCount; c++) {
            campanyList.push(
              CompanyModel.find(
                c != cpuCount
                  ? {
                      createdAt: {
                        $gte: new Date(firstDay.getTime() + folga * (c - 1)),
                        $lt: new Date(firstDay.getTime() + folga * c),
                      },
                    }
                  : {
                      createdAt: {
                        $gte: new Date(firstDay.getTime() + folga * (c - 1)),
                        $lte: new Date(firstDay.getTime() + folga * c),
                      },
                    }
              )
                .lean()
                .populate({
                  path: "laboratories._laboratory",
                  select: "fantasyName cnpj",
                  model: "Laboratory",
                })
               .populate({
                  path: "laboratories._billingDate",
                  select: "period payday",
                  model: "BillingDate",
                })
               .populate({
                  path: "laboratories._price",
                  select: "type fixed percentage clt cnh other",
                  model: "Price",
                })
               .populate({
                  path: "_representative",
                  select: "legalName cnpj name",
                  model: "Representative",
                })
               .then((value) => {
                  return value;
                })
               .catch((errors) => {
                  console.log(errors);
                  return errors;
                })
           );
          }
          Promise.all(campanyList).then(async (response) => {
          
            var List = [];
            
            for await (const iterator of response) {
             List.push(...iterator);
           }
            return res.send(List);
          });
        });
        
