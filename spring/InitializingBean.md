InitializingBean 是在当前 Bean 自己的属性注入完成后，
对当前 Bean 自己执行初始化逻辑（afterPropertiesSet()）。（在这个逻辑里，你当然
可以去使用那些已经注入进来的 Bean，但这只是手段，不是目的）。