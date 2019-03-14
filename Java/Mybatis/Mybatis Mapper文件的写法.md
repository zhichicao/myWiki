# Mybatis Mapper文件的写法



平日工作过程中经常要自己手写mapper文件，首先要建一个mapper的接口类。

比如说一张供应商的表格，要写一个查找供应商的方法，就先建一个mapper类：

```java
package com.tuhu.purchase.mapper.mysql.tuhu_purchase.ext;

import com.tuhu.purchase.domain.mysql.tuhu_purchase.entity.Supplier;
import com.tuhu.purchase.request.supplier.GetSupplierListReqBean;

import java.util.List;

public interface ExternalSupplierMapper {
    List<Supplier> getSupplierList(GetSupplierListReqBean param);
}

```



之后建xml文件，mapper.xml，注意mapper标签写的是上面那个类的路径：

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tuhu.purchase.mapper.mysql.tuhu_purchase.ext.ExternalSupplierMapper">

</mapper>
```



然后将要做的查询语句写在mapper标签里面：



### 查询：

parameterType代表入参的类，resultType代表出参的类

where语句中的like：

```java
<if test="keywordCompanyName != null and keywordCompanyName != ''">
    AND (supplier_name LIKE CONCAT('%',#{keywordCompanyName},'%') OR supplier_short_name LIKE CONCAT('%',#{keywordCompanyName},'%'))
</if>
```

某字段相等equal to：

```java
<if test="majorTypeEnum != null and majorTypeEnum != ''" >
    AND major_class_enum = #{majorTypeEnum}
</if>
```

查询字段在某个list中：

```java
<if test="statusEnumList != null and statusEnumList.size() > 0">
    AND valid_status_enum in
    <foreach collection="statusEnumList" item="item" open="(" close=")" separator=",">
    #{item}
    </foreach>
</if>
```

查询时间范围在signStartDate与signEndDate之间，日期格式是年-月-日

```java
<if test="signStartDate != '' and  signStartDate != null">
    AND ct.`sign_date` >= DATE_FORMAT(#{signStartDate},'%Y-%m-%d')
</if>
<if test="signEndDate != '' and  signEndDate != null">
    AND DATE_FORMAT(#{signEndDate},'%Y-%m-%d') >= ct.`sign_date`
</if>
```



查询供应商列表语句：

```java
    <!--查询供应商列表-->
    <select id="getSupplierList" parameterType="com.tuhu.purchase.request.supplier.GetSupplierListReqBean"
            resultType="com.tuhu.purchase.domain.mysql.tuhu_purchase.entity.Supplier">
        SELECT
        pkid pkid,
        supplier_name supplierName,
        supplier_short_name supplierShortName,
        supplier_type supplierType,
        supplier_type_enum supplierTypeEnum,
        major_class majorClass,
        major_class_enum majorClassEnum,
        subClass subClass,
        subClass_enum subclassEnum,
        province_code provinceCode,
        province_name provinceName,
        city_code cityCode,
        city_name cityName,
        district_code districtCode,
        district_name districtName,
        address address,
        contact_name contactName,
        contact_mobile contactMobile,
        legal_person_name legalPersonName,
        legal_person_mobile legalPersonMobile,
        id_card idCard,
        id_card_attachment idCardAttachment,
        unified_social_credit_code unifiedSocialCreditCode,
        business_license_attachment businessLicenseAttachment,
        bank bank,
        bank_id bankId,
        branch_bank branchBank,
        branch_bank_id branchBankId,
        account_name accountName,
        bank_account bankAccount,
        bank_license_attachment bankLicenseAttachment,
        valid_status validStatus,
        valid_status_enum validStatusEnum,
        work_order_status workOrderStatus,
        work_order_status_enum workOrderStatusEnum,
        remark remark

        FROM
        supplier
        <where>
            and disabled=0
            <if test="keywordCompanyName != null and keywordCompanyName != ''">
                AND (supplier_name LIKE CONCAT('%',#{keywordCompanyName},'%') OR supplier_short_name LIKE CONCAT('%',#{keywordCompanyName},'%'))
            </if>
            <if test="majorTypeEnum != null and majorTypeEnum != ''" >
                AND major_class_enum = #{majorTypeEnum}
            </if>
            <if test="supplierTypeEnum != null and supplierTypeEnum != ''">
                AND supplier_type_enum = #{supplierTypeEnum}
            </if>
            <if test="statusEnumList != null and statusEnumList.size() > 0">
                AND valid_status_enum in
                <foreach collection="statusEnumList" item="item" open="(" close=")" separator=",">
                    #{item}
                </foreach>
            </if>
            <if test="subclassEnum != null and subclassEnum != ''">
                AND subclass_enum = #{subclassEnum}
            </if>
        </where>
        ORDER BY create_time DESC
    </select>
```



查询合同列表的mapper，用了left join：

```java
    <select id="getSubclassExtendPropertyList" parameterType="Integer" resultType="com.tuhu.purchase.domain.mysql.tuhu_purchase.ext.SubclassExtendPropertyDto">
          SELECT  ce.`pkid` AS propertyId,
          ce.`description_name` AS propertyName,
          cs.`necessary` AS necessary,
          cs.`pkid` AS subclassExtendId,
          ce.`sort`
        FROM (select * from  `contract_extend_property` where  `valid_status_enum`=10) AS ce
        LEFT JOIN `contract_subclass_extend`  AS cs
        ON cs.`property_id` = ce.`pkid` and cs.`subclass_id` = #{subclassId}
         ORDER BY ce.`sort` ASC,ce.`pkid` ASC
    </select>
```

查询签署合同的mapper，用了left join、日期判断：



```java
    <select id="getReceiptContractList" parameterType="com.tuhu.purchase.request.contract.GetContractListReqBean"
            resultType="com.tuhu.purchase.response.contract.GetReceiptContractListRespBean">
        SELECT
        ct.`pkid` AS contractId,
        ct.`create_time` AS createDate,
        ct.`subject_company_name` AS subjectCompanyName,
        ct.`contract_code` AS contractCode,
        pList.formatPartners,
        ct.`contract_name` AS contractName,
        cmc.`pkid` as majorClassId,
        cmc.`major_class_name` as majorClassName,
        csc.`pkid` as secondaryClassId,
        csc.`secondary_class_name` as secondaryClassName,
        ct.`effective_date`AS effectiveDate,
        ct.`expiry_date` AS expiryDate,
        ct.`valid_status` AS validStatus,
        ct.`sign_status` AS signStatus
        FROM `contract` AS ct
        LEFT JOIN `contract_config` AS cc ON ct.`pkid` = cc.`contract_id`
        LEFT JOIN `contract_subclass` AS cs ON cc.`subclass_id` = cs.`pkid`
        LEFT JOIN `contract_secondary_class` AS csc ON cs.`secondary_class_pkid` = csc.`pkid`
        LEFT JOIN `contract_major_class` AS cmc ON csc.`major_class_pkid` = cmc.`pkid`
        LEFT JOIN
        (SELECT cp.`contract_id` AS contractId ,GROUP_CONCAT(DISTINCT cp.`partner_name`) AS formatPartners FROM
        `contract_partner` AS cp GROUP BY cp.`contract_id`)
        AS pList
        ON ct.`pkid` = pList.contractId
        where
        ct.`disabled` = 0
        AND ct.`valid_status_enum` != 5
        AND ct.`contract_type_enum` = 5
        <if test="keywordContractCode != '' and keywordContractCode != null">
            AND ct.`contract_code` LIKE CONCAT('%',#{keywordContractCode},'%')
        </if>
        <if test="keywordPartnerName != '' and keywordPartnerName != null">
            AND pList.formatPartners LIKE CONCAT('%',#{keywordPartnerName},'%')
        </if>
        <if test="signStartDate != '' and  signStartDate != null">
            AND ct.`sign_date` >= DATE_FORMAT(#{signStartDate},'%Y-%m-%d')
        </if>
        <if test="signEndDate != '' and  signEndDate != null">
            AND DATE_FORMAT(#{signEndDate},'%Y-%m-%d') >= ct.`sign_date`
        </if>
        <if test="optEmail != '' and  optEmail != null">
            AND ct.`person_in_charge_email` = #{optEmail}
        </if>
        ORDER BY createDate Desc
    </select>
```









