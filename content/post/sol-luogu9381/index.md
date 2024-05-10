---
title: 「那些脑海里最珍贵的」题解
description: 来自 THUPC 2023 决赛的大模拟！
slug: sol-luogu9381
date: 2024-05-10 00:00:05+0800
categories:
    - editorial
tags:
    - 模拟
---

这是笔者自己写的第一篇纯大模拟，因此本题的代码也是目前笔者写过最长的代码。

## How to Code

~~依题意模拟即可。~~

本题除了模拟代码外几乎没有任何思维含量，纯纯的码力题，只要审题足够仔细并且有足够的耐心，A 掉这题是并不困难的。如果写大模拟的经验不是很足的话，看到题目甚至会无从下手（比如我）。这时候就最好从头到尾把题目读一遍，或者多读几遍，然后尽量顺着程序的大致运行过程开始编码。下面就跟着程序运行的逻辑顺序来一步一步实现代码。首先，规定一下本文中某些变量类型的替代称谓（C++11 语法）：

``` cpp
using f64 = double;
using str = std::string;
using MAP_SI = std::unordered_map<str, int>;
using MAP_II = std::unordered_map<int, int>;
```

### 玩家信息输入

首先，几乎可以完全确定的是，题目给定的数据全部都是有用的。所以输入的所有东西都要存储起来。并且，对于本题中这样有关 “队伍” 与 “玩家” 这样阵营分明的设定，我们可以几乎肯定地使用类（结构体）将同一实体有关的数值封装起来。这样就获得了我们的 `PLAYER` 类，用于存储一个玩家的全部有关信息。由于题目有关玩家信息的输入是以玩家为单位的，所以我们在该类中定义一个成员函数 `input()` 用于处理单个玩家信息的输入。当然，我们也需要将题目中涉及到的其他没有被输入的玩家属性存储到这个类中。

``` cpp
int member_cnt[2];

struct PLAYER {
    int team_fr; // 归属队伍
    int team_id; // 归属队伍内编号

    int type; // 类型：0 = weak, 1 = average, 2 = strong
    int lvl; // 等级：1 ~ 100
    f64 atk; // 基础攻击力
    f64 def; // 基础防御力
    int max_hp; // 体力上限
    int act_skl; // 主动技能等级：0 ~ 5
    int psv_skl; // 被动技能等级：0 ~ 5

    int weapon_type; // 武器类型：0 = B, 1 = G, 2 = M
    f64 weapon_atk; // 武器攻击力

    int hp; // 当前体力
    bool status; // 状态：0 = dead, 1 = alive
    f64 skl_boost; // 技能加成

    void input() {
        char player_type[10], weapon_type_t[4];
        scanf("%s Lv=%d maxhp=%d atk=%lf def=%lf skillLv=%d passivesklLv=%d %s weaponatk=%lf\n",
            player_type, &lvl, &max_hp, &atk, &def, &act_skl, &psv_skl, weapon_type_t, &weapon_atk);

        hp = max_hp; // 初始生命值为上限
        status = true; // 初始角色未倒下
        skl_boost = 1; // 初始技能加成为 1
        type = PLAYER_TYPE[str(player_type)]; // 根据输入获取角色类型对应数字
        weapon_type = WEAPON_TYPE[str(weapon_type_t)]; // 根据输入获取武器类型对应数字
    }
} team[2][7];

// ...

int main() {
    scanf("%d %d\n", &member_cnt[0], &member_cnt[1]);

    // 输入南队队员
    for (int i = 1; i <= member_cnt[0]; ++i) {
        team[0][i].input();
        team[0][i].team_fr = 0;
        team[0][i].team_id = i;
    }

    // 输入北队队员
    for (int i = 1; i <= member_cnt[1]; ++i) {
        team[1][i].input();
        team[1][i].team_fr = 1;
        team[1][i].team_id = i;
    }
}
```

为了方便，我们将输入的字符串属性转换成数字，存储在一个常量哈希表中（详见文末代码部分）。

### 获取当前回合的攻击方队员编号

通过阅读题面，我们可以知道，每一轮的攻击方（或技能施放方，以下我们将一回合内主动进行操作的一队统称为攻击方队伍）队伍是根据回合数确定的，攻击方队伍中的攻击者是根据队员编号进行选择的。

为了追踪两队的攻击队员，我们定义数组 `attacker[2]`，分别存储两队当前的攻击队员编号。接着，为了根据题目含义确定每一回合的攻击队员，我们定义 `get_attacker()` 函数：

``` cpp
int attacker[2];

/// @brief 计算每一回合攻击方队伍的攻击者
/// @param t 攻击方队伍的编号
void get_attacker(int t) {
    do {
        ++attacker[t];
        if (attacker[t] > member_cnt[t]) attacker[t] = 1;
    } while (!team[t][attacker[t]].status); // 如果找到未倒下的角色就跳出循环
    // 由于题目保证所有操作合法，所以不用担心死循环
}
```

为了确定每一个回合是哪一队进行攻击，我们在每次循环开头执行以下代码。

``` cpp
int round_cnt;

int main() {
    // ...

    scanf("%d\n", &round_cnt);

    for (int RND = 1; RND <= round_cnt; ++RND) {
        int t_a = (RND % 2) ^ 1; // 计算攻击方队伍
        int t_b = (RND % 2); // 计算防御方队伍

        // 获取攻击方队伍的当前攻击者
        get_attacker(t_a);
        int active = attacker[t_a];
    }
}
```

### 计算伤害

伤害计算是施放攻击对敌人造成影响的重要要素。因此这一部分需要作为铺垫并率先考虑。根据题目中描述：

> 「伤害」，即「体力值」减少的量，由「攻击强度」和受到「伤害」的人的「防御指数」决定。

攻击强度是由攻击队员的各项数值确定的，最终扣血的数值通过受攻击队员的相关防御指数确定。可见，计算完的攻击强度是沟通施放伤害与承受伤害的桥梁。因此我们将计算完的攻击强度作为一个整体，对于单个队员实施扣血操作。为此，我们在 `PLAYER` 类中定义 `deal_damage()` 函数，用于根据当前角色受到攻击的强度来确定扣血的数值。同时，由于攻击强度的计算以及扣血数值的计算都需要用到以队伍为单位的攻击加成和防御加成，所以我们也需要定义这两个浮点型变量 `atk_boost[2]` 和 `def_boost[2]`。由于这两个数值最终是乘到计算中的，所以初始值需要赋值为 1。

在扣血的时候，需要额外注意输出答案中的伤害大小是**原始数值**，但角色实际扣血是不能扣成负数的。其他扣血的地方也是同理。

``` cpp
f64 atk_boost[2] = {1, 1};
f64 def_boost[2] = {1, 1};

struct PLAYER {
    // ...

    void deal_damage(int, int, f64);
}

/// @brief 根据攻击强度对当前角色进行扣血
/// @param t_a 攻击方队伍的编号
/// @param act 攻击者在攻击方队伍内的编号
/// @param eff 攻击强度
void PLAYER::deal_damage(int t_a, int act, f64 eff) {
    if (!status) return; // 保险措施，如果已经倒下就不用考虑了

    int dmg = eff / (def * def_boost[team_fr]); // 计算攻击产生扣血的有效值
    hp -= dmg; // 扣血

    // 如果扣血后角色生命值不高于 0，则该角色倒下
    if (hp <= 0) {
        hp = 0;
        status = false; // 设定角色状态为倒下
    }

    printf("%s %d took %d damage from %s %d -> %d/%d\n",
        (team_fr ? "North" : "South"), team_id, dmg, (t_a ? "North" : "South"), act, hp, max_hp);
}
```

### 攻击

由题面我们可以知道，攻击分四种：普通攻击和三种特殊攻击。阅读题面不难发现，这四种攻击中，只跟攻击队员相关的属性加成，以及跟受攻击队员的躲闪位置有关的方位加成，对于任何一种攻击都是相同的，所以我们先预处理出这一部分属性的加成。普通攻击和 B 特殊攻击比较简单，直接调用受攻击者的 `deal_damage()` 函数，并传入计算好的攻击强度值即可。G 攻击和 M 攻击属于 AoE 攻击，与受攻击队员的站位有关，因此我们先根据题目要求定义两个数组（我不知道为什么我要用一个哈希表）`PLACE[]` 和 `INIT_INDEX[]`，分别存储队员的站位顺序和每一个队员在队伍中的位置编号，这两个数组的信息是互为键值的。由此我们就可以获取 AoE 攻击中受攻击的队员了。接下来依题意模拟即可。

需要注意的是，在 AoE 伤害中，虽然攻击强度中的方位加成是按照主要被攻击者的躲闪方位来计算的，但种族加成是按照受到攻击的实体本身的种族来计算的，所以这一部分需要单独考虑，这也是我把种族加成从预处理中孤立出来的一个主要因素。

``` cpp
const int PLACE[6] = {5, 3, 1, 2, 4, 6};
MAP_II INIT_INDEX = {{1, 2}, {2, 3}, {3, 1}, {4, 4}, {5, 0}, {6, 5}};

/// @brief 实施普通攻击和特殊攻击
/// @param atk_p 攻击位置
/// @param ddg_p 躲闪位置
/// @param act 攻击者在攻击方队伍内的编号
/// @param trg 防御者在防御方队伍内的编号
/// @param t_a 攻击方队伍编号
/// @param t_b 防御方队伍编号
/// @param type 攻击类型，-1 为普通攻击，0~2 为对应的特殊攻击
void perform_attack(int atk_p, int ddg_p, int act, int trg, int t_a, int t_b, int type = -1) {
    // 由于 G 和 M 类型的种族加成对不同目标有不同的取值，所以预处理时不包含种族加成
    f64 eff = team[t_a][act].atk *
        team[t_a][act].weapon_atk *
        team[t_a][act].skl_boost *
        atk_boost[t_a] *
        LOC_BOOST[(atk_p - ddg_p + 6) % 6];

    // 普通攻击
    if (type == -1)
        return team[t_b][trg].deal_damage(t_a, act, eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);

    // B 型特殊攻击
    if (type == 0)
        return team[t_b][trg].deal_damage(t_a, act, 1.25 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);

    // 预处理 G 和 M 类型特殊 AoE 攻击的范围
    int trg_l = -1, trg_r = -1;

    // 西边第一个未倒下的角色
    for (int i = INIT_INDEX[trg] - 1; i >= 0; --i)
        if (team[t_b][PLACE[i]].status) {
            trg_l = PLACE[i];
            break;
        }

    // 东边第一个未倒下的角色
    for (int i = INIT_INDEX[trg] + 1; i < 6; ++i)
        if (team[t_b][PLACE[i]].status) {
            trg_r = PLACE[i];
            break;
        }

    // G 型特殊攻击
    if (type == 1) {
        int cand_cnt = (trg_l != -1) + (trg_r != -1) + 1;
        f64 mul = 1.35 / cand_cnt;

        team[t_b][trg].deal_damage(t_a, act, mul * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);
        if (trg_l != -1) team[t_b][trg_l].deal_damage(t_a, act, mul * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_l].type]);
        if (trg_r != -1) team[t_b][trg_r].deal_damage(t_a, act, mul * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_r].type]);
    }

    // M 型特殊攻击
    else if (type == 2) {
        team[t_b][trg].deal_damage(t_a, act, 1.15 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);
        if (trg_l != -1) team[t_b][trg_l].deal_damage(t_a, act, 0.23 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_l].type]);
        if (trg_r != -1) team[t_b][trg_r].deal_damage(t_a, act, 0.23 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_r].type]);
    }
}

int main() {
    // ...

    for (int RND = 1; RND <= round_cnt; ++RND) {
        // ...

        char action_type[20];
        int trg;

        scanf("%s target=%d ", action_type, &trg);

        // 施放普通攻击
        if (str(action_type) == "Basicattack") {
            int atk_p, ddg_p;
            scanf("atkpos=%d ddgpos=%d \n", &atk_p, &ddg_p);
            perform_attack(atk_p, ddg_p, active, trg, t_a, t_b);
            team[t_a][active].skl_boost = 1; // 施放攻击之后重置技能加成
        }

        // 施放特殊攻击
        else if (str(action_type) == "Specialattack") {
            int atk_p, ddg_p;
            scanf("atkpos=%d ddgpos=%d \n", &atk_p, &ddg_p);
            perform_attack(atk_p, ddg_p, active, trg, t_a, t_b, team[t_a][active].weapon_type);
            team[t_a][active].skl_boost = 1; // 施放攻击之后重置技能加成
        }

        putchar('\n');
    }
}
```

### 释放主动技能

由于技能部分涉及到了很多常数，所以我们不如先把所有数值存起来。这个时候就体现出将字符串属性存储为整数的优越之处了。注意我的存法中，还包含了技能等级为 0 时的加成，虽然可能用不到，但还是存一下比较保险。

``` cpp
const f64 ACT_SKL_MUL[3][6] = {
    {0.00, 0.10, 0.12, 0.15, 0.17, 0.20},
    {0.00, 0.06, 0.07, 0.08, 0.09, 0.10},
    {1.00, 2.10, 2.17, 2.24, 2.32, 2.40}
};

const f64 PSV_SKL_MUL[3][6] = {
    {0.00, 0.013, 0.016, 0.019, 0.022, 0.025},
    {0.00, 0.01, 0.02, 0.03, 0.04, 0.05},
    {0.00, 0.01, 0.02, 0.03, 0.04, 0.05}
};
```

有关主动技能，有几点值得注意。

第一是 Weak 类型的主动技能，技能释放的对象是本队成员，所以在传入参数的时候，目标队伍需要设定为本队。在回复生命的时候，需要额外注意输出答案中的回复量大小是**原始数值**，但角色实际回复生命是不能超过上限的。其他回复生命的地方也是同理。

第二是 Average 类型的主动技能，会给对方施加 DoT 持续伤害并在**技能释放者所在队伍的主动回合的回合末**进行结算。由于 DoT 伤害一旦施加，就只与被施加的敌方队员有关了，所以我们在 `PLAYER` 类中对有关 DoT 伤害的信息进行存储，并定义函数 `DOT()`，在每一个生效回合末激活。

```cpp
struct PLAYER {
    // ...

    int dot_dmg; // 持续伤害大小
    int dot_rounds; // 持续伤害层数

    // ...

    void DOT();
} team[2][7];

/// @brief DoT，即 Damage over Time，处理当前角色受到的 Average 类型的持续伤害
void PLAYER::DOT() {
    if (!status) return; // 保险措施，如果已经倒下就不用考虑了

    if (dot_rounds) { // 如果持续伤害层数不为 0
        --dot_rounds, hp -= dot_dmg; // 则触发一次持续伤害并减一层层数

        // 同上
        if (hp <= 0) {
            hp = 0;
            status = false;
        }

        void input() {
            // ...
            dot_rounds = 0; // 初始持续伤害剩余层数为 0
            dot_dmg = 0; // 初始持续伤害大小为 0
        }

        printf("%s %d took %d damage from skill -> %d/%d\n",
            (team_fr ? "North" : "South"), team_id, dot_dmg, hp, max_hp);
    }
}
```

那么接下来就可以写处理主动技能释放的函数了。

```cpp
/// @brief 触发主动技能
/// @param act 释放技能者在释放技能方队伍内的编号
/// @param trg 被释放技能者在被释放技能方队伍内的编号
/// @param t_a 释放技能方队伍的编号
/// @param t_b 被释放技能队伍的编号
void activate_active_skill(int act, int trg, int t_a, int t_b) {
    // 获取技能类型名以便输出
    str skl_type;

    if (team[t_a][act].type == 0) skl_type = "Weak";
    else if (team[t_a][act].type == 1) skl_type = "Average";
    else if (team[t_a][act].type == 2) skl_type = "Strong";

    printf("%s %d applied %s skill to %s %d\n",
        (t_a ? "North" : "South"), act, skl_type.c_str(), (t_b ? "North" : "South"), trg);

    // 对于 Weak 类型的主动技能
    if (team[t_a][act].type == 0) {
        int tmp = team[t_b][trg].hp;
        int heal = ACT_SKL_MUL[0][team[t_a][act].act_skl] * team[t_b][trg].max_hp; // 计算回复量
        team[t_b][trg].hp += heal; // 回复
        if (team[t_b][trg].hp > team[t_b][trg].max_hp) // 如果回复后超过生命上限
            team[t_b][trg].hp = team[t_b][trg].max_hp; // 则将生命值改为上限

        if (tmp != team[t_b][trg].hp) // 如果生命值有所变动，就需要输出
            printf("%s %d recovered +%d hp -> %d/%d\n",
                (t_b ? "North" : "South"), trg, heal, team[t_b][trg].hp, team[t_b][trg].max_hp);
    }

    // 对于 Average 类型的主动技能
    else if (team[t_a][act].type == 1) {
        team[t_b][trg].dot_rounds = 3; // 设持续伤害持续 3 层
        team[t_b][trg].dot_dmg = ACT_SKL_MUL[1][team[t_a][act].act_skl] * team[t_b][trg].max_hp; // 计算持续伤害大小
    }

    // 对于 Strong 类型的主动技能
    else if (team[t_a][act].type == 2)
        team[t_b][trg].skl_boost = ACT_SKL_MUL[2][team[t_a][act].act_skl]; // 更改角色的技能加成
}

int main() {
    // ...

    for (int RND = 1; RND <= round_cnt; ++RND) {
        // ...

        // 施放主动技能
        else if (str(action_type) == "Skill") // 只有 Average 类型的主动技能是对敌方的
            activate_active_skill(active, trg, t_a, (team[t_a][active].type == 1 ? t_b : t_a));

        // 对防守方队伍中所有收到持续伤害的队员结算持续伤害（同队施放的持续伤害只在同队为攻击方的回合内生效）
        for (int i = 1; i <= member_cnt[t_b]; ++i)
            team[t_b][i].DOT();

        putchar('\n');
    }
}
```

### 计算并施加被动技能

被动技能是本题主要的坑点之一，虽然题目已经说明，但还是很容易漏写。在角色倒下时，该角色给己方队伍贡献的被动技能数值需要重新计算。每回合开始时，还需要处理 Weak 类型的被动技能对己方队伍成员的生命值回复。因此我们写一个专门的函数 `reapply_passive_skills()`，在每次需要重新计算被动技能时，整体全部统计一遍。（反正数据范围小，复杂度再高也没什么问题。）

``` cpp
/// @brief 在每回合开始时及有角色倒下后重新计算被动伤害
/// @param t 默认为 -1，有角色倒下；如果输入为 0 或 1，则代表 Weak 类型的被动技能生效的团队
void reapply_passive_skills(int t = -1) {
    if (t != -1) { // 如果是开局，则需要使 Weak 类型的被动效果生效
        f64 weak = 0; // 统计 Weak 被动叠加效果层数
        for (int i = 1; i <= member_cnt[t]; ++i)
            if (team[t][i].status &&
                team[t][i].type == 0) // 对于当前队伍中所有还活着的 Weak 类型角色
                weak += PSV_SKL_MUL[0][team[t][i].psv_skl]; // 根据其等级叠加效果加成

        weak = std::min(weak, 0.05); // 考虑效果加成上限

        for (int i = 1; i <= member_cnt[t]; ++i) {
            if (!team[t][i].status) continue; // 已经倒下的角色就不用考虑了

            int tmp = team[t][i].hp;
            int heal = weak * team[t][i].max_hp; // 计算回复量
            team[t][i].hp += heal; // 回复
            if (team[t][i].hp > team[t][i].max_hp) // 如果回复后超过生命上限
                team[t][i].hp = team[t][i].max_hp; // 则将生命值更改为上限

            if (tmp != team[t][i].hp) // 如果生命值有所变动，就需要输出
                printf("%s %d recovered +%d hp -> %d/%d\n",
                    (t ? "North" : "South"), i, heal, team[t][i].hp, team[t][i].max_hp);
        }
    }

    // 对于其他两种类型，只需要在角色倒下后考虑被动技能分配情况即可
    for (int t = 0; t < 2; ++t) {
        f64 average = 0, strong = 0; // 统计被动效果叠加层数
        for (int i = 1; i <= member_cnt[t]; ++i)
            if (team[t][i].status) // 对于当前队伍中所有还活着的对应类型队员
                if (team[t][i].type == 1) // 统计 Average 被动叠加层数
                    average += PSV_SKL_MUL[1][team[t][i].psv_skl];
                else if (team[t][i].type == 2) // 统计 Strong 被动叠加层数
                    strong += PSV_SKL_MUL[2][team[t][i].psv_skl];

        // 根据上限进行相应调整
        average = std::min(average, 0.1);
        strong = std::min(strong, 0.1);

        // 增加对应的防御加成、攻击加成的值
        def_boost[t] = average + 1;
        atk_boost[t] = strong + 1;
    }
}
```

将这个函数添加到每回合开始时以及有角色倒下后即可。

### 其他细节

比如回合末信息的输出和最后赢家的判定。这些小细节就请看下面的完整代码了。

## Code

以下代码建议使用 Visual Studio Code 或者其他功能较丰富的编辑器打开预览，因为我在部分函数上添加了 Doxygen 注释，以便增强代码可读性，即使已经没有什么可读性可言了。但一定不要复制粘贴提交，一定要自己写一遍！

``` cpp
#include <cstdio>
#include <string>
#include <unordered_map>

#define endl '\n'

using f64 = double;
using str = std::string;
using MAP_SI = std::unordered_map<str, int>;
using MAP_II = std::unordered_map<int, int>;

// 固定的加成数值
const f64 LOC_BOOST[6] = {1.25, 1.00, 0.75, 0.00, 0.75, 1.00};
const f64 TYPE_BOOST[3][3] = {{1.0, 0.9, 1.1}, {1.1, 1.0, 0.9}, {0.9, 1.1, 1.0}};
const int PLACE[6] = {5, 3, 1, 2, 4, 6};

const f64 ACT_SKL_MUL[3][6] = {
    {0.00, 0.10, 0.12, 0.15, 0.17, 0.20},
    {0.00, 0.06, 0.07, 0.08, 0.09, 0.10},
    {1.00, 2.10, 2.17, 2.24, 2.32, 2.40}
};

const f64 PSV_SKL_MUL[3][6] = {
    {0.00, 0.013, 0.016, 0.019, 0.022, 0.025},
    {0.00, 0.01, 0.02, 0.03, 0.04, 0.05},
    {0.00, 0.01, 0.02, 0.03, 0.04, 0.05}
};

f64 atk_boost[2] = {1, 1};
f64 def_boost[2] = {1, 1};
MAP_SI WEAPON_TYPE = {{"B", 0}, {"G", 1}, {"M", 2}};
MAP_SI PLAYER_TYPE = {{"Weak", 0}, {"Average", 1}, {"Strong", 2}};
MAP_II INIT_INDEX = {{1, 2}, {2, 3}, {3, 1}, {4, 4}, {5, 0}, {6, 5}};

int member_cnt[2];
int round_cnt;
int attacker[2];

struct PLAYER {
    int team_fr; // 归属队伍
    int team_id; // 归属队伍内编号

    int type; // 类型：0 = weak, 1 = average, 2 = strong
    int lvl; // 等级：1 ~ 100
    f64 atk; // 基础攻击力
    f64 def; // 基础防御力
    int max_hp; // 体力上限
    int act_skl; // 主动技能等级：0 ~ 5
    int psv_skl; // 被动技能等级：0 ~ 5

    int weapon_type; // 武器类型：0 = B, 1 = G, 2 = M
    f64 weapon_atk; // 武器攻击力

    int hp; // 当前体力
    bool status; // 状态：0 = dead, 1 = alive
    f64 skl_boost; // 技能加成

    int dot_dmg; // 持续伤害大小
    int dot_rounds; // 持续伤害层数

    void input() {
        char player_type[10], weapon_type_t[4];
        scanf("%s Lv=%d maxhp=%d atk=%lf def=%lf skillLv=%d passivesklLv=%d %s weaponatk=%lf\n",
            player_type, &lvl, &max_hp, &atk, &def, &act_skl, &psv_skl, weapon_type_t, &weapon_atk);

        hp = max_hp; // 初始生命值为上限
        status = true; // 初始角色未倒下
        skl_boost = 1; // 初始技能加成为 1
        type = PLAYER_TYPE[str(player_type)]; // 根据输入获取角色类型对应数字
        weapon_type = WEAPON_TYPE[str(weapon_type_t)]; // 根据输入获取武器类型对应数字
        dot_rounds = 0; // 初始持续伤害剩余层数为 0
        dot_dmg = 0; // 初始持续伤害大小为 0
    }

    void deal_damage(int, int, f64);
    void DOT();
} team[2][7];

/// @brief 在每回合开始时及有角色倒下后重新计算被动伤害
/// @param t 默认为 -1，有角色倒下；如果输入为 0 或 1，则代表 Weak 类型的被动技能生效的团队
void reapply_passive_skills(int t = -1) {
    if (t != -1) { // 如果是开局，则需要使 Weak 类型的被动效果生效
        f64 weak = 0; // 统计 Weak 被动叠加效果层数
        for (int i = 1; i <= member_cnt[t]; ++i)
            if (team[t][i].status &&
                team[t][i].type == 0) // 对于当前队伍中所有还活着的 Weak 类型角色
                weak += PSV_SKL_MUL[0][team[t][i].psv_skl]; // 根据其等级叠加效果加成

        weak = std::min(weak, 0.05); // 考虑效果加成上限

        for (int i = 1; i <= member_cnt[t]; ++i) {
            if (!team[t][i].status) continue; // 已经倒下的角色就不用考虑了

            int tmp = team[t][i].hp;
            int heal = weak * team[t][i].max_hp; // 计算回复量
            team[t][i].hp += heal; // 回复
            if (team[t][i].hp > team[t][i].max_hp) // 如果回复后超过生命上限
                team[t][i].hp = team[t][i].max_hp; // 则将生命值更改为上限

            if (tmp != team[t][i].hp) // 如果生命值有所变动，就需要输出
                printf("%s %d recovered +%d hp -> %d/%d\n",
                    (t ? "North" : "South"), i, heal, team[t][i].hp, team[t][i].max_hp);
        }
    }

    // 对于其他两种类型，只需要在角色倒下后考虑被动技能分配情况即可
    for (int t = 0; t < 2; ++t) {
        f64 average = 0, strong = 0; // 统计被动效果叠加层数
        for (int i = 1; i <= member_cnt[t]; ++i)
            if (team[t][i].status) // 对于当前队伍中所有还活着的对应类型队员
                if (team[t][i].type == 1) // 统计 Average 被动叠加层数
                    average += PSV_SKL_MUL[1][team[t][i].psv_skl];
                else if (team[t][i].type == 2) // 统计 Strong 被动叠加层数
                    strong += PSV_SKL_MUL[2][team[t][i].psv_skl];

        // 根据上限进行相应调整
        average = std::min(average, 0.1);
        strong = std::min(strong, 0.1);

        // 增加对应的防御加成、攻击加成的值
        def_boost[t] = average + 1;
        atk_boost[t] = strong + 1;
    }
}

/// @brief 根据攻击强度对当前角色进行扣血
/// @param t_a 攻击方队伍的编号
/// @param act 攻击者在攻击方队伍内的编号
/// @param eff 攻击强度
void PLAYER::deal_damage(int t_a, int act, f64 eff) {
    if (!status) return; // 保险措施，如果已经倒下就不用考虑了

    int dmg = eff / (def * def_boost[team_fr]); // 计算攻击产生扣血的有效值
    hp -= dmg; // 扣血

    // 如果扣血后角色生命值不高于 0，则该角色倒下
    if (hp <= 0) {
        hp = 0;
        status = false; // 设定角色状态为倒下
        reapply_passive_skills(); // 重新考虑被动技能
    }

    printf("%s %d took %d damage from %s %d -> %d/%d\n",
        (team_fr ? "North" : "South"), team_id, dmg, (t_a ? "North" : "South"), act, hp, max_hp);
}

/// @brief DoT，即 Damage over Time，处理当前角色受到的 Average 类型的持续伤害
void PLAYER::DOT() {
    if (!status) return; // 保险措施，如果已经倒下就不用考虑了

    if (dot_rounds) { // 如果持续伤害层数不为 0
        --dot_rounds, hp -= dot_dmg; // 则触发一次持续伤害并减一层层数

        // 同上
        if (hp <= 0) {
            hp = 0;
            status = false;
            reapply_passive_skills();
        }

        printf("%s %d took %d damage from skill -> %d/%d\n",
            (team_fr ? "North" : "South"), team_id, dot_dmg, hp, max_hp);
    }
}

/// @brief 计算每一回合攻击方队伍的攻击者
/// @param t 攻击方队伍的编号
void get_attacker(int t) {
    do {
        ++attacker[t];
        if (attacker[t] > member_cnt[t]) attacker[t] = 1;
    } while (!team[t][attacker[t]].status); // 如果找到未倒下的角色就跳出循环
    // 由于题目保证所有操作合法，所以不用担心死循环
}

/// @brief 触发主动技能
/// @param act 释放技能者在释放技能方队伍内的编号
/// @param trg 被释放技能者在被释放技能方队伍内的编号
/// @param t_a 释放技能方队伍的编号
/// @param t_b 被释放技能队伍的编号
void activate_active_skill(int act, int trg, int t_a, int t_b) {
    // 获取技能类型名以便输出
    str skl_type;

    if (team[t_a][act].type == 0) skl_type = "Weak";
    else if (team[t_a][act].type == 1) skl_type = "Average";
    else if (team[t_a][act].type == 2) skl_type = "Strong";

    printf("%s %d applied %s skill to %s %d\n",
        (t_a ? "North" : "South"), act, skl_type.c_str(), (t_b ? "North" : "South"), trg);

    // 对于 Weak 类型的主动技能
    if (team[t_a][act].type == 0) {
        int tmp = team[t_b][trg].hp;
        int heal = ACT_SKL_MUL[0][team[t_a][act].act_skl] * team[t_b][trg].max_hp; // 计算回复量
        team[t_b][trg].hp += heal; // 回复
        if (team[t_b][trg].hp > team[t_b][trg].max_hp) // 如果回复后超过生命上限
            team[t_b][trg].hp = team[t_b][trg].max_hp; // 则将生命值改为上限

        if (tmp != team[t_b][trg].hp) // 如果生命值有所变动，就需要输出
            printf("%s %d recovered +%d hp -> %d/%d\n",
                (t_b ? "North" : "South"), trg, heal, team[t_b][trg].hp, team[t_b][trg].max_hp);
    }

    // 对于 Average 类型的主动技能
    else if (team[t_a][act].type == 1) {
        team[t_b][trg].dot_rounds = 3; // 设持续伤害持续 3 层
        team[t_b][trg].dot_dmg = ACT_SKL_MUL[1][team[t_a][act].act_skl] * team[t_b][trg].max_hp; // 计算持续伤害大小
    }

    // 对于 Strong 类型的主动技能
    else if (team[t_a][act].type == 2)
        team[t_b][trg].skl_boost = ACT_SKL_MUL[2][team[t_a][act].act_skl]; // 更改角色的技能加成
}

/// @brief 实施普通攻击和特殊攻击
/// @param atk_p 攻击位置
/// @param ddg_p 躲闪位置
/// @param act 攻击者在攻击方队伍内的编号
/// @param trg 防御者在防御方队伍内的编号
/// @param t_a 攻击方队伍编号
/// @param t_b 防御方队伍编号
/// @param type 攻击类型，-1 为普通攻击，0~2 为对应的特殊攻击
void perform_attack(int atk_p, int ddg_p, int act, int trg, int t_a, int t_b, int type = -1) {
    // 由于 G 和 M 类型的种族加成对不同目标有不同的取值，所以预处理时不包含种族加成
    f64 eff = team[t_a][act].atk *
        team[t_a][act].weapon_atk *
        team[t_a][act].skl_boost *
        atk_boost[t_a] *
        LOC_BOOST[(atk_p - ddg_p + 6) % 6];

    // 普通攻击
    if (type == -1)
        return team[t_b][trg].deal_damage(t_a, act, eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);

    // B 型特殊攻击
    if (type == 0)
        return team[t_b][trg].deal_damage(t_a, act, 1.25 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);

    // 预处理 G 和 M 类型特殊 AoE 攻击的范围
    int trg_l = -1, trg_r = -1;

    // 西边第一个未倒下的角色
    for (int i = INIT_INDEX[trg] - 1; i >= 0; --i)
        if (team[t_b][PLACE[i]].status) {
            trg_l = PLACE[i];
            break;
        }

    // 东边第一个未倒下的角色
    for (int i = INIT_INDEX[trg] + 1; i < 6; ++i)
        if (team[t_b][PLACE[i]].status) {
            trg_r = PLACE[i];
            break;
        }

    // G 型特殊攻击
    if (type == 1) {
        int cand_cnt = (trg_l != -1) + (trg_r != -1) + 1;
        f64 mul = 1.35 / cand_cnt;

        team[t_b][trg].deal_damage(t_a, act, mul * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);
        if (trg_l != -1) team[t_b][trg_l].deal_damage(t_a, act, mul * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_l].type]);
        if (trg_r != -1) team[t_b][trg_r].deal_damage(t_a, act, mul * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_r].type]);
    }

    // M 型特殊攻击
    else if (type == 2) {
        team[t_b][trg].deal_damage(t_a, act, 1.15 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg].type]);
        if (trg_l != -1) team[t_b][trg_l].deal_damage(t_a, act, 0.23 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_l].type]);
        if (trg_r != -1) team[t_b][trg_r].deal_damage(t_a, act, 0.23 * eff * TYPE_BOOST[team[t_a][act].type][team[t_b][trg_r].type]);
    }
}

int main() {
    scanf("%d %d\n", &member_cnt[0], &member_cnt[1]);

    // 输入南队队员
    for (int i = 1; i <= member_cnt[0]; ++i) {
        team[0][i].input();
        team[0][i].team_fr = 0;
        team[0][i].team_id = i;
    }

    // 输入北队队员
    for (int i = 1; i <= member_cnt[1]; ++i) {
        team[1][i].input();
        team[1][i].team_fr = 1;
        team[1][i].team_id = i;
    }

    scanf("%d\n", &round_cnt);

    for (int RND = 1; RND <= round_cnt; ++RND) {
        int t_a = (RND % 2) ^ 1; // 计算攻击方队伍
        int t_b = (RND % 2); // 计算防御方队伍

        // 对攻击方队伍施放 Weak 类型被动技能
        reapply_passive_skills(t_a);

        // 获取攻击方队伍的当前攻击者
        get_attacker(t_a);
        int active = attacker[t_a];

        char action_type[20];
        int trg;

        scanf("%s target=%d ", action_type, &trg);

        // 施放普通攻击
        if (str(action_type) == "Basicattack") {
            int atk_p, ddg_p;
            scanf("atkpos=%d ddgpos=%d \n", &atk_p, &ddg_p);
            perform_attack(atk_p, ddg_p, active, trg, t_a, t_b);
            team[t_a][active].skl_boost = 1; // 施放攻击之后重置技能加成
        }

        // 施放特殊攻击
        else if (str(action_type) == "Specialattack") {
            int atk_p, ddg_p;
            scanf("atkpos=%d ddgpos=%d \n", &atk_p, &ddg_p);
            perform_attack(atk_p, ddg_p, active, trg, t_a, t_b, team[t_a][active].weapon_type);
            team[t_a][active].skl_boost = 1; // 施放攻击之后重置技能加成
        }

        // 施放主动技能
        else if (str(action_type) == "Skill") // 只有 Average 类型的主动技能是对敌方的
            activate_active_skill(active, trg, t_a, (team[t_a][active].type == 1 ? t_b : t_a));

        // 对防守方队伍中所有收到持续伤害的队员结算持续伤害（同队施放的持续伤害只在同队为攻击方的回合内生效）
        for (int i = 1; i <= member_cnt[t_b]; ++i)
            team[t_b][i].DOT();

        // 先输出北队，再输出南队
        for (int t = 1; ~t; --t) {
            printf("%s: ", t ? "North" : "South");
            for (int i = 0; i < 6; i++)
                if (PLACE[i] <= member_cnt[t]) // 如果队伍不足 6 人就不输出对应位置的生命值
                    printf("%d/%d ", team[t][PLACE[i]].hp, team[t][PLACE[i]].max_hp);
            putchar('\n');
        }

        putchar('\n');
    }

    // 统计是否有队伍获胜
    int alive_cnt[2] = {0, 0};

    for (int t = 0; t < 2; ++t)
        for (int i = 1; i <= member_cnt[t]; ++i)
            if (team[t][i].status || team[t][i].hp)
                ++alive_cnt[t];

    if (!alive_cnt[0])
        puts("Team North won.");
    else if (!alive_cnt[1])
        puts("Team South won.");

    return fflush(stdout), 0;
}
```

## Tips

写这道题让我获得了许多写模拟的心得，下面简单说几条我觉得比较重要的。

- 对于纯模拟题，一定要保证关键信息的可获取性。简单来说，在任何环境下，你都一定要保留至少一种查询到你所需要的信息的方式。它决定了我们需要维护什么东西。
- 保证代码可读。平时写 OI 题那种只有一个字母的变量名尽量少出现，多写注释，要让自己时刻清楚自己在写什么代码、实现什么功能，不被大量的信息冲昏头脑。
- 善用 Ctrl/Cmd + F 查找。像本题这种，同样一种属性可能在很多地方被影响或者影响其他数值的情况，最好用浏览器或 PDF 阅读器的查找功能找到某一个属性在题目表述中所有出现的地方，确保自己考虑周全了。

## Epilogue

感谢你看到这里。这份代码或许不是最为简洁、最有效率的一份代码，但我依然希望你在完成这道题、阅读这篇题解的过程中有所收获。毕竟，正如出题人提供的的大样例成为我 AC 这道题的道路上一份有力的援助一样，写出这道题也一定是我们锻炼自己编码能力和调试能力、最终成为一名优秀的程序员的路途中一次十分有价值的经历吧。与你共勉。
