---
title: mybatis关联查询association
date: 2020-03-29 14:11:25
updated: 2020-03-29 14:11:25
tags: 
- mybatis
- association
---

##### mybatis的association两种方式

1. 嵌套的resultMap

   <!--more-->

   bean如下：

   ```java
   
   public class Product {
       private String spId;
       private String systemCode;
       private List<Plan> plans;
   
       public String getSystemCode() {
           return systemCode;
       }
   
       public void setSystemCode(String systemCode) {
           this.systemCode = systemCode;
       }
   
       public String getSpId() {
           return spId;
       }
   
       public void setSpId(String spId) {
           this.spId = spId;
       }
   
       public List<Plan> getPlans() {
           return plans;
       }
   
       public void setPlans(List<Plan> plans) {
           this.plans = plans;
       }
   }
   
   public class Plan {
       private String statusName;
       private String cardType;
       private List<Card> cards;
   
       public String getStatusName() {
           return statusName;
       }
   
       public void setStatusName(String statusName) {
           this.statusName = statusName;
       }
   
       public String getCardType() {
           return cardType;
       }
   
       public void setCardType(String cardType) {
           this.cardType = cardType;
       }
   
       public List<Card> getCards() {
           return cards;
       }
   
       public void setCards(List<Card> cards) {
           this.cards = cards;
       }
   }
   
   public class Card {
       private String cardTypeId;
       private String cardTypeValue;
   
       public String getCardTypeId() {
           return cardTypeId;
       }
   
       public void setCardTypeId(String cardTypeId) {
           this.cardTypeId = cardTypeId;
       }
   
       public String getCardTypeValue() {
           return cardTypeValue;
       }
   
       public void setCardTypeValue(String cardTypeValue) {
           this.cardTypeValue = cardTypeValue;
       }
   }
   
   ```

   xml如下

   ```xml
   		<resultMap type="Product" id="ProductResult">
           <result property="spId" column="sp_id"/>
           <result property="systemCode" column="system_code"/>
           <association property="plans" resultMap="PlanResult"/>
       </resultMap>
   
       <resultMap type="Plan" id="PlanResult">
           <result property="cardType" column="card_type"/>
           <result property="statusName" column="status_name"/>
           <association property="cards" resultMap="CardResult"/>
       </resultMap>
   
       <resultMap type="Card" id="CardResult">
           <result property="cardTypeId" column="card_type_id"/>
           <result property="cardTypeValue" column="card_type_value"/>
       </resultMap>
       <select id="listProduct" resultMap="ProductResult">
           select t1.sp_id,
                  t1.system_code,
                  t2.status_name,
                  t2.card_type,
                  t3.card_type_id,
                  t3.card_type_value
           from zy_cfg_system_prod_rel t1
           left join zy_cfg_prod_settle_plan t2 on t1.sp_id=t2.sp_id
           left join zy_cfg_card_config_inf t3 on t2.card_type=t3.card_type_id;
       </select>
   ```

2. 嵌套的select语句

   javabean的结构还是一样，配置文件如下：

   ```xml
   		<resultMap type="Product" id="ProductResult">
           <result property="spId" column="sp_id"/>
           <result property="systemCode" column="system_code"/>
     <!--column如果想传入多个参数的话，可以 column="{spId=sp_id,systemCode=system_code}" 来传参-->
           <association property="plans" column="sp_id" select="listPlan"/>
       </resultMap>
   
       <resultMap type="Plan" id="PlanResult">
           <result property="cardType" column="card_type"/>
           <result property="statusName" column="status_name"/>
           <association property="cards" column="card_type" select="listCard"/>
       </resultMap>
   
       <resultMap type="Card" id="CardResult">
           <result property="cardTypeId" column="card_type_id"/>
           <result property="cardTypeValue" column="card_type_value"/>
       </resultMap>
   
   		<select id="listProduct" resultMap="ProductResult">
           select sp_id,system_code from zy_cfg_system_prod_rel
       </select>
   
       <select id="listPlan" resultMap="PlanResult">
           select status_name,card_type from zy_cfg_prod_settle_plan where sp_id=#{spId}
       </select>
   
       <select id="listCard" resultMap="CardResult">
           select card_type_id,card_type_value from zy_cfg_card_config_inf where card_type_id=#{cardType}
       </select>
   
   ```

   ##### 总结

   尽量不要用嵌套的select语句，优先使用嵌套的resultMap。

