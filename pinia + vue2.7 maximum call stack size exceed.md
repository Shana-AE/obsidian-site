pinia中使用composition api 拆分store时遇到问题，xxxById的数据一修改就会报maximum call stack size exceed，报错为vue中出现，callstack为isArray及_traverse  不知道具体原因，但是后续尝试将子composition拆分到一个单独的store中此问题消失，不清楚具体情况， 猜测和根store的某些数据冲突。