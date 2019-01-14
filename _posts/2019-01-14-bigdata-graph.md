---
layout: post
title:  "关于无向图中的路径搜索试验"
categories: 大数据
tags:  图计算
---

* content
{:toc}

## 博客链接
https://www.cnblogs.com/hlongch/p/5892594.html

## 问题分析
在运营商的传输电路中，每个县区会有两台ASG传输设备用于用户接入侧到骨干传输网的连接，在ASG下面又挂有多个CSG设备，从而形成县区的传输电路。如果某个CSG设备脱管，则可能会影响到其他CSG设备，从而影响多个小区上网。如何根据脱管的CSG设备计算出受影响的网络设备？
本质上来说这是一个无向图的路径搜索问题，即当有脱管设备时，将此设备与其他节点的关系删除，遍历所有设备，如果设备可以找到路径到达其中任何一台ASG设备，则认为此设备没有问题，否则认为此节点也受影响

## 代码
根据网上代码例子，用现实的数据测试了一下计算效果，速度上还可以接受，本例子中节点数35个。本例中提供了一个方法封装。

~~~
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Stack;

/**
 * 无向无权无环图<br/>
 * 寻找起点到终点的所有路径 定义
 * 根据节点间的关系及脱管节点，给出所有受影响的节点数
 */
public class GrfAllEdge {
// 图的顶点总数
private int total;
// 各顶点基本信息
private String[] nodes;
// 脱管的节点
private List<String> nodesUn;
// asg1在节点中的索引
private int asg1Index;
// asg2在节点中的索引
private int asg2Index;
// 图的邻接矩阵
private int[][] matirx;

/**
 * 
 * @param total     节点总数
 * @param nodesA    节点数组
 * @param nodesZ    每节节点关联的节点目标关系
 * @param asg1Index asg1节点在节点数组中的索引位置
 * @param asg2Index asg2节点在节点数组中的索引位置
 * @param nodesUn   脱管的节点列表
 */
public GrfAllEdge(int total, String[] nodesA, String[] nodesZ, int asg1Index, int asg2Index, List<String> nodesUn) {
this.total = total;
this.nodes = nodesA;
this.matirx = new int[total][total];
this.asg1Index = asg1Index;
this.asg2Index = asg2Index;
this.nodesUn = nodesUn;
for (int i = 0; i < total; i++) {
String node = nodesZ[i];
for (int j = 0; j < total; j++) {
if (node == nodesA[j]) {
this.matirx[i][j] = 1;
this.matirx[j][i] = 1;
break;
}
}
}
// 脱管节点与其他节点设置无关系
for (int i = 0; i < total; i++) {
if (this.nodesUn.contains(nodes[i])) {
for (int j = 0; j < total; j++) {
this.matirx[j][i] = 0;
this.matirx[i][j] = 0;
}
}
}
// 节点自身不会相连
for (int i = 0; i < this.total; i++) {
this.matirx[i][i] = 0;
}
// 设置两个asg节点间无关联
this.matirx[asg1Index][asg2Index] = 0;
this.matirx[asg2Index][asg1Index] = 0;
}

/**
 * 获取每条路径的经过节点
 * 
 * @param stack
 * @param k
 * @return
 */
private String getStackLink(Stack<Integer> stack, int k) {
StringBuilder sb = new StringBuilder();
for (Integer i : stack) {
sb.append(this.nodes[i] + ",");
}
sb.append(this.nodes[k] + ",");
return sb.toString();
}

/**
 * 寻找起点到终点的所有路径
 * 
 * @param underTop 紧挨着栈顶的下边的元素
 * @param goal     目标
 * @param stack
 */
private void dfsStack(int underTop, int goal, Stack<Integer> stack, List<String> lines) {
if (stack.isEmpty()) {
return;
}

// 访问栈顶元素，但不弹出
int k = stack.peek().intValue();
// 紧挨着栈顶的下边的元素
// int uk = underTop;

if (k == goal) {
// System.out.print("\n起点与终点不能相同");
return;
}

// 对栈顶的邻接点依次递归调用，进行深度遍历
for (int i = 0; i < this.total; i++) {
// 有边，并且不在左上到右下的中心线上
if (this.matirx[k][i] == 1 && k != i) {
// 排除环路
if (stack.contains(i)) {
// 由某顶点A，深度访问其邻接点B时，由于是无向图，所以存在B到A的路径，在环路中，我们要排除这种情况
// 严格的请，这种情况也是一个环
// if (i != uk) {
// System.out.print("\n有环:");
// this.printStack(stack, i);
// }
continue;
}

// 打印路径
if (i == goal) {
//System.out.print("\n路径:");
lines.add(getStackLink(stack, i));
continue;
}
// 深度遍历
stack.push(i);
dfsStack(k, goal, stack, lines);
}
}
stack.pop();
}

/**
 * 打印节点关系矩阵图
 */
public void printMatrix() {

System.out.println("----------------- matrix -----------------");
for (int i = 0; i < this.total; i++) {
System.out.print(this.nodes[i] + "|");
}
System.out.println();
for (int i = 0; i < this.total; i++) {
System.out.print(" " + this.nodes[i] + "|");
for (int j = 0; j < this.total; j++) {
System.out.print(this.matirx[i][j] + "-");
}
System.out.print("\n");
}
System.out.println("----------------- matrix -----------------");
}

/**
 * 
 * @return 返回受影响的节点，不包括asg节点，如果ASG节点不正常，则此方法不适用
 */
public List<String> unreachableNodeAll() {
List<String> unreachableNodeList = new ArrayList<String>();
// 查询节点关联到主ASG节点
for (int i = 0; i < this.total; i++) {
int origin = i;
int goal1 = this.asg1Index;
Stack<Integer> stack1 = new Stack<Integer>();
List<String> lines1 = new ArrayList<String>();
stack1.push(origin);
this.dfsStack(-1, goal1, stack1, lines1);

int goal2 = this.asg2Index;
Stack<Integer> stack2 = new Stack<Integer>();
List<String> lines2 = new ArrayList<String>();
stack2.push(origin);
this.dfsStack(-1, goal2, stack2, lines2);
if (lines1.size() < 1 && lines2.size() < 1 && i != goal1 && i != goal2) {
unreachableNodeList.add(this.nodes[i]);
}
}
return unreachableNodeList;
}

public static void main(String[] args) {
String[] nodesA = new String[] { "冀州县局2_CSG", "冀州大寨模块_CSG", "冀州大寨模块局_CSG", "冀州南大方基站_CSG", "冀州南良村基站_CSG",
"冀州大寨基站（电信）_CSG", "冀州宾馆基站_CSG", "冀州三基站_CSG", "冀州魏屯基站_CSG", "冀州王家宜子基站_CSG", "冀州炉具市场基站_CSG",
"冀州李桃园基站_CSG", "冀州温泉基站_CSG", "冀州大常庄基站_CSG", "冀州法院家属院基站_CSG", "冀州华阳基站_CSG", "冀州县局_CX600-X8_ASG_2",
"冀州周村_CSG", "冀州周村基站_CSG", "冀州新寨模块_CSG", "冀州北漳淮基站（电信）_CSG", "冀州北漳淮_CSG", "冀州北内漳基站_CSG", "冀州吴吕基站_CSG",
"冀州南午村基站_CSG", "冀州三里庄基站_CSG", "冀州中心局基站_CSG", "冀州中心局1_ATN950B_CSG", "冀州彭村基站_CSG", "冀州吉爽暖气（艺科）基站_CSG",
"冀州县局_CX600-X8_ASG_1", "冀州温州大厦_CSG", "冀州碧水康庭基站_CSG", "冀州张家宜子基站_CSG", "冀州岳家庄基站_CSG" };
String[] nodesZ = new String[] { "冀州县局_CX600-X8_ASG_1", "冀州县局_CX600-X8_ASG_1", "冀州大寨模块_CSG", "冀州大寨模块_CSG",
"冀州大寨模块_CSG", "冀州大寨模块_CSG", "冀州县局_CX600-X8_ASG_1", "冀州县局_CX600-X8_ASG_1", "冀州县局_CX600-X8_ASG_1",
"冀州魏屯基站_CSG", "冀州县局_CX600-X8_ASG_1", "冀州炉具市场基站_CSG", "冀州炉具市场基站_CSG", "冀州县局_CX600-X8_ASG_1",
"冀州县局_CX600-X8_ASG_1", "冀州县局_CX600-X8_ASG_1", "冀州县局_CX600-X8_ASG_1", "冀州县局_CX600-X8_ASG_2", "冀州周村_CSG",
"冀州周村_CSG", "冀州新寨模块_CSG", "冀州北漳淮基站（电信）_CSG", "冀州北漳淮_CSG", "冀州北内漳基站_CSG", "冀州吴吕基站_CSG",
"冀州县局_CX600-X8_ASG_2", "冀州县局_CX600-X8_ASG_2", "冀州中心局基站_CSG", "冀州中心局1_ATN950B_CSG", "冀州彭村基站_CSG",
"冀州吉爽暖气（艺科）基站_CSG", "冀州中心局基站_CSG", "冀州县局_CX600-X8_ASG_2", "冀州碧水康庭基站_CSG", "冀州县局_CX600-X8_ASG_2" };

// 获取ASG的索引
int a1Index = -1;
int a2Index = -1;
for (int i = 0; i < nodesA.length; i++) {
if (nodesA[i].contains("ASG_1")) {
a1Index = i;
}
if (nodesA[i].contains("ASG_2")) {
a2Index = i;
}
}
// 添加脱管节点
// List<String> nodesUn = Arrays.asList("冀州大常庄基站_CSG", "冀州周村_CSG","冀州吉爽暖气（艺科）基站_CSG", "冀州中心局基站_CSG");
List<String> nodesUn = Arrays.asList("冀州中心局1_ATN950B_CSG");
// 根据节点数组、节点目标关系数组、asg1的索引位置、asg2的索引位置、脱管的节点名称列表构造图关系实例
GrfAllEdge grf = new GrfAllEdge(nodesA.length, nodesA, nodesZ, a1Index, a2Index, nodesUn);
// 打印节点关系矩阵图
grf.printMatrix();
// 获取受影响的节点列表
List<String> list = grf.unreachableNodeAll();
System.out.println("脱管节点列表:" + nodesUn);
System.out.println("受影响的节点列表:" + list);

}

}
~~~

## 效果展示
下面红色小圈标志的是脱管节点，黄色节点是另外受到影响的节点。
![](https://raw.githubusercontent.com/shockw/shockw.github.io/master/img/20190114/DDE31575-2774-4593-8EFB-B25DC7094500.png)

