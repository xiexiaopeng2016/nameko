### 3.1 融拓基础模块
定义业务模块共用的模型和函数.

#### 3.1.1 eSIM抽象模型
其他模型通过继承这个模型来拥有这些字段. 模型名称 `abstract.esim`, 描述 `eSIM抽象模型`.

#### 3.1.1.1 字段

* `billing_cycle` `计费周期`, 类型`Integer`
* `billing_uom_id` `计费周期单位`, 类型`Many2one`, 关联对象`uom.uom`, 删除时`restrict`
* `data_size` `套餐高速流量`, 类型`Integer`
* `lot_id` `批次号`, 类型`Many2one`, 关联对象`stock.production.lot`, 删除时`restrict`, 过滤条件`[('product_id', '=', product_id)]`
* `mo_qty` `上行短信数量`, 类型`Integer`
* `mt_qty` `下行短信数量`, 类型`Integer`
* `name` `名称`, 类型`Char`, 必填`True`, 复制`False`
* `product_id` `产品`, 类型`Many2one`, 关联对象`product.product`, 删除时`restrict`
* `rate_limit` `限速`, 类型`Integer`
* `used_data_size` `已用流量`, 类型`Integer`
* `used_mo_qty` `已用上行短信数量`, 类型`Integer`
* `used_mt_qty` `已用下行短信数量`, 类型`Integer`
* `warning_value` `告警值`, 类型`Float`
* `warning_value_type` `告警值类型`, 类型`Selection`, 选项`[('number', '剩余流量'), ('percentage', '剩余百分比')]`

#### 3.1.1.2 方法
* 无

#### 3.1.2 客户拥有的卡
模型名称 `esim.card`, 描述 `客户拥有的卡`.

#### 3.1.2.1 字段

* `card_pool_id` `套餐池`, 类型`Many2one`, 关联对象`esim.card.pool`, 删除时`restrict`
* `has_data_group` `组成套餐池`, 类型`Boolean`
* `online_state` `在线状态`, 类型`Selection`, 选项`[('on', '在线'), ('off', '不在线')]`
* `partner_id` `客户`, 类型`Many2one`, 关联对象`res.partner`, 删除时`restrict`
* `state` `状态`, 类型`Selection`, 选项`[('power-off', '未开卡'), ('power-on', '已开卡'), ('using', '使用中'), ('suspended', '已暂停'), ('scraped', '已报废')]`
* `validity_period` `eSIM有效期`, 类型`Float`

#### 3.1.2.2 方法
* 无

#### 3.1.3 套餐池
模型名称 `esim.card.pool`, 描述 `套餐池`.

#### 3.1.3.1 字段

* `esim_card_ids` `eSIM卡`, 类型`One2many`, 关联对象`esim.card`, 反向字段`card_pool_id`, 复制`True`
* `esim_card_qty` `设备数`, 类型`Integer`, 计算方法`_compute_esim_card_qty`, 存储`True`, 只读`True`, 复制`False`
* `max_data_qty` `单设备流量上限`, 类型`Integer`
* `name` `名称`, 类型`Char`, 必填`True`, 复制`False`
* `total_data_qty` `总流量`, 类型`Integer`
* `usable_data_qty` `可用流量`, 类型`Integer`, 计算方法`_compute_usable_data_qty`, 存储`True`, 只读`True`, 复制`False`
* `used_data_qty` `已用流量`, 类型`Integer`

#### 3.1.3.2 方法
* `_compute_esim_card_qty` 
    ```
    @api.depends('esim_card_ids')
    def _compute_esim_card_qty(self):
        """计算套餐池管理的设备数量"""
        for record in self:
            record.esim_card_qty = len(record.esim_card_ids)
    ```
* `_compute_usable_data_qty` 
    ```
    @api.depends('total_data_qty', 'used_data_qty')
    def _compute_usable_data_qty(self):
        """计算套餐池剩余流量"""
        for record in self:
            record.usable_data_qty = record.total_data_qty - record.used_data_qty
    ```

#### 3.1.4 eSIM卡套餐
模型名称 `esim.data.plan`, 描述 `eSIM卡套餐`.

#### 3.1.4.1 字段

* `state` `状态`, 类型`Selection`, 选项`[('using', '使用中'), ('stoped', '已关闭')]`, 默认值 `using`
* `supported_country_ids` `支持的国家`, 类型`One2many`, 关联对象`esim.supported.country`, 反向字段`data_plan_id`, 复制`True`

#### 3.1.4.2 方法
* 无

#### 3.1.5 运营商服务
模型名称 `esim.operator.service`, 描述 `运营商服务`.

#### 3.1.5.1 字段

* `country_id` `国家`, 类型`Many2one`, 关联对象`res.country`, 删除时`restrict`
* `data` `密文`, 类型`Char`
* `data_plan_id` `eSIM卡套餐`, 类型`Many2one`, 关联对象`esim.data.plan`, 删除时`restrict`
* `end_time` `结束时间`, 类型`Datetime`
* `imis` `副号IMSI`, 类型`Char`
* `open_time` `开通时间`, 类型`Datetime`
* `operator_id` `运营商`, 类型`Many2one`, 关联对象`res.operator`, 删除时`restrict`
* `service_type` `服务类别`, 类型`Selection`, 选项`[('data', '流量')]`

#### 3.1.5.2 方法
* 无

#### 3.1.6 套餐支持的国家和运营商
模型名称 `esim.supported.country`, 描述 `套餐支持的国家和运营商`.

#### 3.1.6.1 字段

* `country_id` `国家`, 类型`Many2one`, 关联对象`res.country`, 删除时`restrict`
* `esim_card_id` `设备`, 类型`Many2one`, 关联对象`esim.card`, 删除时`restrict`
* `operator_id` `运营商`, 类型`Many2one`, 关联对象`res.operator`, 删除时`restrict`

#### 3.1.6.2 方法
* 无

#### 3.1.7 res.partner
模型名称 `res.partner`.

#### 3.1.7.1 字段

* `esim_ids` `eSIM卡`, 类型`One2many`, 关联对象`esim.card`, 反向字段`partner_id`, 复制`True`

#### 3.1.7.2 方法
* 无

