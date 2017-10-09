# hierarchical-data
层级数据的动态分层


动态生成节点层级位置  避免节点间连线的覆盖

要求：
    1.节点数目不是很多  
    2.具有不同类型的节点  type -> 1:输入  2:任意 3:任意 4:输出  连线从输入开始  输出结束 
    3.数据类型 edges ->  {a:[b,c],b:[c,d],c:[f]}   nodes -> {a:{...},b:{...},c:{...},...,f:{...}}



思路:
    1.创建一个10x10的二维数组，用于定位节点
    for(let index = 0,array = []; index < 10; index++) {
        array.push(new Array(10));
    }

    2.区分出 3种节点（输入，输出，任意）  并放入二维数组中(基础是5x5的二维数组)
    let inputs = [],outputs = [],others = [];
    Object.values(nodes).forEach((node) => {
        if(node.type === 1) {
            inputs.push(node.name);
        }else if(node.type === 4){
            outputs.push(node.name)
        }else {
            others.push(node.name);
        }
    });

    let location = {},otherIndex = 0,_others = [],maxX = inputs.length - 1;//maxX 横向层数
    //输入节点放入二维数组，与输入节点相连的节点提前放入
    inputs.forEach((input,index) => {
        location[input] = {y:0,x:index};
        array[0][index] = location[input];
        _others = _others.concat(edges[input] || []);
    });
    //除输出节点  其他的也需要放入二维数组
    while(_others.length > 0) {
        let _others2 = [];
        _others.forEach(otherNode => {
            if(!location[otherNode] && !outputs.includes(otherNode)) {
                location[otherNode] = {x: otherIndex++};
            }
            _others2 = _others2.concat(edges[otherNode] || []);
        });
        _others = _others2;
    }

    others.forEach((other,index) => {
        const _index = location[other].x;
        const _y = 1 + Math.floor(_index/5)
        const _x = _index%5;
        location[other] = {y:_y,x:_x};
        array[_y][_x] = location[other];
        if( _x > maxX) {
            maxX = _x;
        }
    });

    let maxY = 1 + Math.floor((others.length - 1)/5);//maxY 竖向层数

    3.根据edges数据生成需要校验的边数组 仅起点:A 仅终点:B 起点且终点：C  需要判断 AAB、ABB、ACB 这三种 避免边重合//outputs  先排除outputs计算出层级位置  再加入outputs再计算出最终的层级位置
    madeJudgeArray(edges,outputs) {
        let judgeList = [],_judgeList = [];
        let typeABCJudgeList = [];
        let _edges = {};//用于获取AAB类型

        //获取ABB类型的边
        for(let key in edges) {
            let value = edges[key];
            let index = 0;
            for(; index < value.length - 1; index++) {
                if(outputs.includes(value[index])) {
                    continue;
                }

                //获取ACB类型的边
                if(edges[value[index]]) {
                    edges[value[index]].forEach((nodeName) => {
                        typeABCJudgeList.push([key,value[index],nodeName]);
                    });
                }

                if(!_edges[value[index]]) {
                    _edges[value[index]] = [];
                }

                _edges[value[index]].push(key);
                for(let next = index + 1;next < value.length; next++) {
                    if(outputs.includes(value[next])) {
                        continue;
                    }
                    judgeList.push([key,value[index],value[next]]);
                }
                
            }

            if(!_edges[value[index]]) {
                _edges[value[index]] = [];
            }

            _edges[value[index]].push(key);
        }

        //获取AAB类型的边
        for(let _key in _edges) {
            let _value = _edges[_key];
            let _index = 0;
            
            if(outputs.includes(_key)) {
                continue;
            }

            for(; _index < _value.length - 1; _index++) {
                if(outputs.includes(_value[_index])) {
                    continue;
                }

                for(let _next = _index + 1;_next < _value.length; _next++) {
                    if(outputs.includes(_value[_next])) {
                        continue;
                    }
                    _judgeList.push([_key,_value[_index],_value[_next]]);
                }
                
            }
        }

        return {type_AAB_ABB:judgeList.concat(_judgeList),type_ABC:typeABCJudgeList};
    }

    4.判断边是否重合并定位
    let judge = this.madeJudgeArray(edges,outputs);

    function isOK(judge){
        for(let index = 0;index < judge.type_AAB_ABB.length;index++) {
            let list = judge.type_AAB_ABB[index];
            let source = location[list[0]];
            let other1 = location[list[1]];
            let other2 = location[list[2]];
            //判断 AAB 或者 ABB 格式的3个点是否有重合
            if(source.y === other1.y && source.y === other2.y) {
                if(source.x > other1.x && source.x > other2.x) {
                    array[source.y][source.x] = undefined;
                    while(!!array[++source.y][source.x]) {
                    }
                    array[source.y][source.x] = source;
                    if(source.y > maxY) {
                        maxY = source.y;
                    }
                }else if(source.x < other1.x && source.x < other2.x) {
                    array[source.y][source.x] = undefined;
                    while(!!array[++source.y][source.x]) {
                    }
                    array[source.y][source.x] = source;
                    if(source.y > maxY) {
                        maxY = source.y;
                    }
                }

                return false;
            }else if(source.x === other1.x && source.x === other2.x){
                if(source.y > other1.y && source.y > other2.y) {
                    array[source.y][source.x] = undefined;
                    while(!!array[source.y][++source.x]) {
                    }
                    array[source.y][source.x] = source;
                    if(source.x > maxX) {
                        maxX = source.x;
                    }                
                }else if(source.y < other1.y && source.y < other2.y) {
                    if(other1.y > other2.y) {
                        array[other1.y][other1.x] = undefined;
                        while(!!array[other1.y][++other1.x]) {
                        }
                        array[other1.y][other1.x] = other1;
                        if(other1.x > maxX) {
                            maxX = other1.x;
                        }
                    }else {
                        array[other2.y][other2.x] = undefined;
                        while(!!array[other2.y][++other2.x]) {
                        }
                        array[other2.y][other2.x] = other2;
                        if(other2.x > maxX) {
                            maxX = other2.x;
                        }
                    }
                }

                return false;
            }else if(other1.x - other2.x === source.x - other1.x 
                    && other1.y - other2.y === source.y - other1.y){
                if(source.y > other2.y) {
                    array[source.y][source.x] = undefined;
                    while(!!array[source.y][++source.x]) {
                    }
                    array[source.y][source.x] = source;
                    if(source.x > maxX) {
                        maxX = source.x;
                    }
                }else {
                    array[other2.y][other2.x] = undefined;
                    while(!!array[other2.y][++other2.x]) {
                    }
                    array[other2.y][other2.x] = other2;
                    if(other2.x > maxX) {
                        maxX = other2.x;
                    }
                }

                return false;
            }else if(other2.x - other1.x === source.x - other2.x 
                && other2.y - other1.y === source.y - other2.y){
                if(source.y > other1.y) {
                    array[source.y][source.x] = undefined;
                    while(!!array[source.y][++source.x]) {
                    }
                    array[source.y][source.x] = source;
                    if(source.x > maxX) {
                        maxX = source.x;
                    }
                }else {
                    array[other1.y][other1.x] = undefined;
                    while(!!array[other1.y][++other1.x]) {
                    }
                    array[other2.y][other1.x] = other1;
                    if(other1.x > maxX) {
                        maxX = other1.x;
                    }
                }

                return false;
            }
        }

        //判断 ACB 格式的3个点是否有重合
        for(let index = 0;index < judge.type_ABC.length;index++) {
            let list = judge.type_ABC[index];
            let source1 = location[list[0]];
            let source2 = location[list[1]];
            let target = location[list[2]];
            if(source1.y === source2.y && source1.y === target.y) {
                if((source2.x > source1.x && source2.x > target.x)
                    || (source2.x < source1.x && source2.x < target.x)) {
                    array[source2.y][source2.x] = undefined;
                    while(!!array[++source2.y][source2.x]) {
                    }
                    array[source2.y][source2.x] = source2;
                    if(source2.y > maxY) {
                        maxY = source2.y;
                    }
                }

                return false;
            }else if(source1.x === source2.x && source1.x === target.x){
                if((source2.y > source1.y && source2.y > target.y)
                    || (source2.y < source1.y && source2.y < target.y)) {
                    array[source2.y][source2.x] = undefined;
                    while(!!array[source2.y][++source2.x]) {
                    }
                    array[source2.y][source2.x] = source2;
                    if(source2.x > maxX) {
                        maxX = source2.x;
                    }
                }

                return false;
            }else if(source1.x - source2.x === target.x - source1.x 
                && source1.y - source2.y === target.y - source1.y){
                    array[source2.y][source2.x] = undefined;
                    while(!!array[source2.y][++source2.x]) {
                    }
                    array[source2.y][source2.x] = source2;
                    if(source2.x > maxX) {
                        maxX = source2.x;
                    }

                    return false;
            }else if(target.x - source2.x === source1.x - target.x 
                && target.y - source2.y === source1.y - target.y){
                    array[source2.y][source2.x] = undefined;
                    while(!!array[source2.y][++source2.x]) {
                    }
                    array[source2.y][source2.x] = source2;
                    if(source2.x > maxX) {
                        maxX = source2.x;
                    }

                    return false;
            }

        }

        return true;
        
    }

    while(!isOK(judge)) {}

    outputs.forEach((output,index) => {
        location[output] = {y:maxY+1,x:index};
        array[maxY+1][index] = location[output];
    });

    judge = this.madeJudgeArray(edges,[]);
    while(!isOK(judge)) {}
