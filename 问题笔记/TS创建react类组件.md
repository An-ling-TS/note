# TS创建react类组件
[toc]
## 问题
使用TS创建React类组件
使用Reac.Component时报错
```
interface IProps {

}
interface State {
    t: string;
}
export default class GameOne extends React.Component<IProps, State> {
    constructor(props) {
        super(props);
        this.state = {
            t: '2'
        };
    }
    render() {
        return (
            <div>232323</div>
        )
    }
}
```
## 原因
未知
## 解决方案
将React.Component改为Component