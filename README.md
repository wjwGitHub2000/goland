func GoodsTask() {
	cn := cron.New(cron.WithSeconds())
	_, err := cn.AddFunc("*/30 * * * * *", func() {
		all, err := mysql.GoodsGetAll()
		if err != nil {
			log.Fatal("mysql查询失败")
		}
		for _, v := range all {
			b := es.GoodsAdd(strconv.Itoa(int(v.ID)), v)
			if !b {
				log.Fatal("es同步失败")
			}
			for i := 0; i < v.GoodsStock; i++ {
				bs := redis.GoodsRPush(strconv.Itoa(int(v.ID)))
				if !bs {
					log.Fatal("redis商品同步失败")
				}
			}

		}

		fmt.Println("执行成功一次")
	})
	if err != nil {
		panic(err)
	}
	cn.Start()
	select {}
}
