====================
Conversion rule info
====================

.. toctree::

WdExtensions
==============================================

WdExtensions - is a help java class for working with dictionaries inside XSLT map.
In order to apply it, the following namespace has to be added:

.. code:: xml

    <?xml version='1.0'?>
    <xsl:stylesheet version="1.0"
                    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
                    xmlns:saxon="http://saxon.sf.net/"
                    xmlns:wdExtensions="java:com.whitedoc.xslt.extensions.WdExtensions"
                    exclude-result-prefixes="saxon wdExtensions">

There are 2 static methods:

| 1. wdExtensions:getValueFromDictionary(String dictionaryUuid, String columnByUuid, String valueToFind, String columnToFind)
| Can be used to find a value in column

| 2. wdExtensions:getRecordUuidByValueFromDictionary(String dictionaryUuid, String columnUuid, String valueToFind)
| Can be used to find UUID of dictionary record

Conversion rule example for outgoing documents
==============================================

.. code:: xml

<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:saxon="http://saxon.sf.net/" xmlns:wdExtensions="java:com.whitedoc.xslt.extensions.WdExtensions" exclude-result-prefixes="saxon wdExtensions">
	<xsl:output indent="yes"/>

	<!--Корневой темплейт задается отдельно, для оптимизации, чтобы в каждой выборке не писать ORDER/-->
	<xsl:template match="/">
		<xsl:apply-templates select="ORDER"/>
	</xsl:template>

	<xsl:template match="ORDER">
		<envelope>
			<!--UUID шаблона нужно скопировать с формы на портале-->
			<xsl:attribute name="templateUuid" select="'fdf13b93-a82f-4d48-885f-ddbadf365174'"/>
			<!--UUID версии шаблона-->
			<xsl:attribute name="templateVersion" select="'7b627762-fba6-4b60-8875-da3cb8d55269'"/>
			<info>
				<subject>
					<xsl:value-of select="concat('Замовлення на постачання № ',NUMBER)"/>
				</subject>
				<message>
					<xsl:choose>
						<xsl:when test="DOCTYPE='O'">Оригінал замовлення</xsl:when>
						<xsl:when test="DOCTYPE='R'">Заміна замовлення</xsl:when>
						<xsl:when test="DOCTYPE='D'">Видалення замовлення</xsl:when>
						<xsl:when test="DOCTYPE='F'">Фіктивність замовлення</xsl:when>
						<xsl:when test="DOCTYPE='PO'">Попереднє замовлення</xsl:when>
						<xsl:when test="DOCTYPE='OS'">Замовлення на послугу / маркетинг</xsl:when>
						<xsl:otherwise>
							<xsl:value-of select="'Оригінал замовлення'"/>
						</xsl:otherwise>
					</xsl:choose>
				</message>
			</info>
			<state>
				<message>
					<xsl:value-of select="INFO"/>
				</message>
				<date>
					<xsl:value-of select="current-date() "/>
				</date>
				<roleId>
				</roleId>
			</state>
			<documents>
				<document>
					<!--Задание UUID конвертируемого документа, смотрим в схеме конвертации перед настройкой интеграции-->
					<!--Атрибуты необходимо задавать специальной командой, тогда будет отрабатывать корректно логика при структурировании карты-->
					<xsl:attribute name="id" select="'cd511754-cfd1-49a3-aae4-aab212d83cea'"/>
					<field>
						<!--Номер заказа в системе покупателя-->
						<xsl:attribute name="name" select="'NUMBER'"/>
						<xsl:value-of select="NUMBER"/>
					</field>
					<field>
						<!--Дата заказа в системе покупателя-->
						<xsl:attribute name="name" select="'DATE'"/>
						<xsl:value-of select="DATE"/>
					</field>
					<field>
						<!--Дата доставки-->
						<xsl:attribute name="name" select="'DELIVERYDATE'"/>
						<xsl:value-of select="DELIVERYDATE"/>
					</field>
					<!--Проверка на заполнение необходима в тегах, которые могут быть не заполнены покупателем.-->
					<xsl:if test="CURRENCY">
						<field>
							<!--Валюта-->
							<xsl:attribute name="name" select="'CURRENCY'"/>
							<xsl:value-of select="CURRENCY"/>
						</field>
					</xsl:if>
					<xsl:if test="string-length(CAMPAIGNNUMBER)!=0">
						<field>
							<!--Номер договора-->
							<xsl:attribute name="name" select="'CAMPAIGNNUMBER'"/>
							<xsl:value-of select="CAMPAIGNNUMBER"/>
						</field>
					</xsl:if>
					<xsl:if test="string-length(INFO)!=0">
						<!--Доп. информация, иногда в этом теге передается информация, которую необходимо обрабатывать-->
						<field>
							<xsl:attribute name="name" select="'INFO'"/>
							<xsl:value-of select="INFO"/>
						</field>
					</xsl:if>
					<xsl:if test="string-length(DOCTYPE)!=0">
						<!--Тип документа: O - оригінал, R - заміна, D - видалення, F - фіктивність замовлення, PO - попереднє замовлення, OS - замовлення на послугу / маркетинг-->
						<field>
							<xsl:attribute name="name" select="'DOCTYPE'"/>
							<xsl:choose>
								<xsl:when test="DOCTYPE='O'">Оригінал замовлення</xsl:when>
								<xsl:when test="DOCTYPE='R'">Заміна замовлення</xsl:when>
								<xsl:when test="DOCTYPE='D'">Видалення замовлення</xsl:when>
								<xsl:when test="DOCTYPE='F'">Фіктивність замовлення</xsl:when>
								<xsl:when test="DOCTYPE='PO'">Попереднє замовлення</xsl:when>
								<xsl:when test="DOCTYPE='OS'">Замовлення на послугу / маркетинг</xsl:when>
								<xsl:otherwise>
									<xsl:value-of select="DOCTYPE"/>
								</xsl:otherwise>
							</xsl:choose>
						</field>
					</xsl:if>
					<field>
						<!--Поставщик(продавац)-->
						<xsl:attribute name="name" select="'SUPPLIER'"/>
						<xsl:attribute name="recordUuid" select="wdExtensions:getRecordUuidByValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/SUPPLIER)"/>
						<xsl:value-of select="HEAD/SUPPLIER"/>
					</field>
					<field>
						<!--Покупатель-->
						<xsl:attribute name="name" select="'BUYER'"/>
						<xsl:attribute name="recordUuid" select="wdExtensions:getRecordUuidByValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/BUYER)"/>
						<xsl:value-of select="HEAD/BUYER"/>
					</field>
					<field>
						<!--Точка доставки-->
						<xsl:attribute name="name" select="'DELIVERYPLACE'"/>
						<xsl:attribute name="recordUuid" select="wdExtensions:getRecordUuidByValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/DELIVERYPLACE)"/>
						<xsl:value-of select="HEAD/DELIVERYPLACE"/>
					</field>

					<xsl:if test="HEAD/INVOICEPARTNER">
						<field>
							<!--Плательщик, этот тег в шаблонах необходимо делать не обязательным и он также может совпадать с покупателем.-->
							<xsl:attribute name="name" select="'INVOICEPARTNER'"/>
							<xsl:value-of select="HEAD/INVOICEPARTNER"/>
						</field>
					</xsl:if>

					<!--<field name="SUPPLIERNAME" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/SUPPLIER, '7cdb31bb-7747-4219-9df9-d62e1e99e40b')"/>
					<field name="SUPPLIERINDEX" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/SUPPLIER, '73cdf547-36a9-44b2-9de8-668e430ca4e9')"/>
					<field name="SUPPLIERCITY" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/SUPPLIER, '3e8830d6-beb2-4da2-8f67-d983ad6774ce')"/>
					<field name="SUPPLIERADDRESS" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/SUPPLIER, '55e0ae28-eebc-43f2-8ebf-88af76a0ce24')"/>-->

					<!--<field name="BUYERNAME" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/BUYER, '7cdb31bb-7747-4219-9df9-d62e1e99e40b')"/>
					<field name="BUYERINDEX" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/BUYER, '73cdf547-36a9-44b2-9de8-668e430ca4e9')"/>
					<field name="BUYERCITY" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/BUYER, '3e8830d6-beb2-4da2-8f67-d983ad6774ce')"/>
					<field name="BUYERADDRESS" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/BUYER, '55e0ae28-eebc-43f2-8ebf-88af76a0ce24')"/>-->

					<!--<field name="DELIVERYPLACENAME" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/DELIVERYPLACE, '7cdb31bb-7747-4219-9df9-d62e1e99e40b')"/>
					<field name="DELIVERYPLACEINDEX" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/DELIVERYPLACE, '73cdf547-36a9-44b2-9de8-668e430ca4e9')"/>
					<field name="DELIVERYPLACECITY" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/DELIVERYPLACE, '3e8830d6-beb2-4da2-8f67-d983ad6774ce')"/>
					<field name="DELIVERYPLACEADDRESS" value="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/DELIVERYPLACE, '55e0ae28-eebc-43f2-8ebf-88af76a0ce24')"/>-->

					<field>
						<!--Отправитель-->
						<xsl:attribute name="name" select="'SENDER'"/>
						<xsl:value-of select="HEAD/SENDER"/>
					</field>
					<field>
						<!--Получатель-->
						<xsl:attribute name="name" select="'RECIPIENT'"/>
						<xsl:value-of select="HEAD/RECIPIENT"/>
					</field>
					<field>
						<!--ID транзакції-->
						<xsl:attribute name="name" select="'EDIINTERCHANGEID'"/>
						<xsl:value-of select="HEAD/EDIINTERCHANGEID"/>
					</field>
					<fieldgroup name="POSITION">
						<!--Подбор товарных строк осуществляется в отдельном темплейте, т.к. строк может быть неограниченное кол-во-->
						<xsl:apply-templates select="HEAD/POSITION"/>
					</fieldgroup>
					<field name="POSITIONSCOUNT">
						<xsl:value-of select="count(HEAD/POSITION)"/>
					</field>
					<field name="SUMORDEREDQUANTITY">
						<xsl:value-of select="format-number(sum(HEAD/POSITION/ORDEREDQUANTITY),'#.###')"/>
					</field>
					<xsl:if test="HEAD/POSITION/ORDERPRICE">
						<field name="POSITIONSAMOUNT">
							<xsl:value-of select="format-number(sum(HEAD/POSITION/ORDEREDQUANTITY*HEAD/POSITION/ORDERPRICE),'#.##')"/>
						</field>
					</xsl:if>
					<xsl:if test="HEAD/POSITION/PRICEWITHVAT">
						<field name="TOTALAMOUNT">
							<xsl:value-of select="format-number(sum(HEAD/POSITION/ORDEREDQUANTITY*HEAD/POSITION/PRICEWITHVAT),'#.##')"/>
						</field>
					</xsl:if>
					<xsl:if test="HEAD/POSITION/ORDERPRICE and HEAD/POSITION/PRICEWITHVAT">
						<field name="VATSUM">
							<xsl:value-of select="format-number(sum(HEAD/POSITION/ORDEREDQUANTITY*HEAD/POSITION/PRICEWITHVAT)-sum(HEAD/POSITION/ORDEREDQUANTITY*HEAD/POSITION/ORDERPRICE),'#.##')"/>
						</field>
					</xsl:if>
				</document>
			</documents>
			<flow>
				<roles>
					<!--Отправитель-->
					<role>
						<xsl:attribute name="id" select="'f9378c46-5dfe-484a-b985-5a157d238b5c'"/>
						<xsl:attribute name="mailboxUuid" select="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/BUYER, '1ddf9e46-49f5-41a2-b6f5-7c3015bc4505')"/>

						<!--<xsl:attribute name="mailboxUuid" select="'19fecc4c-92a3-4cd7-b64a-eeacec8b84d7'"/>-->
					</role>
					<!--Получатель-->
					<role>
						<xsl:attribute name="id" select="'d01dd50c-8e56-432e-9194-6dd6ac6200c8'"/>
						<xsl:attribute name="mailboxUuid" select="wdExtensions:getValueFromDictionary('5bc5be5a-751d-4ae3-8ad8-3a6ac1ab71c8', '4f68fbf0-d78d-4aef-9192-bccd0c8d6011', HEAD/SUPPLIER, '1ddf9e46-49f5-41a2-b6f5-7c3015bc4505')"/>
						<!--<xsl:attribute name="mailboxUuid" select="'3b49d365-5c96-4320-81bf-4b96cfacfcfc'"/>-->
					</role>
				</roles>
			</flow>
		</envelope>
	</xsl:template>


	<xsl:template match="POSITION">
		<!--автоматический счетчик товарных позиций в заказе, начиная с 0-->
		<fieldset index="{position() - 1}">
			<field>
				<!--Порядковый номер товара в заказе-->
				<xsl:attribute name="name" select="'POSITIONNUMBER'"/>
				<xsl:value-of select="POSITIONNUMBER"/>
			</field>
			<field>
				<!--Штрих-код-->
				<xsl:attribute name="name" select="'PRODUCT'"/>
				<xsl:value-of select="PRODUCT"/>
			</field>
			<xsl:if test="PRODUCTIDBUYER">
				<field>
					<!--Артикул покупателя, может не передаваться в исключительных случаях-->
					<xsl:attribute name="name" select="'PRODUCTIDBUYER'"/>
					<xsl:value-of select="PRODUCTIDBUYER"/>
				</field>
			</xsl:if>
			<xsl:if test="PRODUCTIDSUPPLIER">
				<field>
					<!--Артикул поставщика, передаётся редко-->
					<xsl:attribute name="name" select="'PRODUCTIDSUPPLIER'"/>
					<xsl:value-of select="PRODUCTIDSUPPLIER"/>
				</field>
			</xsl:if>
			<xsl:if test="BUYERPARTNUMBER">
				<field>
					<!--Номер партии покупателя-->
					<xsl:attribute name="name" select="'BUYERPARTNUMBER'"/>
					<xsl:value-of select="BUYERPARTNUMBER"/>
				</field>
			</xsl:if>
			<xsl:if test="ORDEREDQUANTITY">
				<field>
					<!--Заказанное кол-во-->
					<xsl:attribute name="name" select="'ORDEREDQUANTITY'"/>
					<xsl:value-of select="ORDEREDQUANTITY"/>
				</field>
			</xsl:if>
			<xsl:if test="ORDERUNIT">
				<field>
					<!--Единица измерения-->
					<xsl:attribute name="name" select="'ORDERUNIT'"/>
					<xsl:attribute name="value" select="ORDERUNIT"/>
					<xsl:attribute name="recordUuid" select="wdExtensions:getRecordUuidByValueFromDictionary('165dbb79-6ffa-40d9-b813-317a46c0cc88', '307cc5c5-7479-4cf3-8e80-0f0468557ab7', ORDERUNIT)"/>
					<xsl:value-of select="wdExtensions:getValueFromDictionary('165dbb79-6ffa-40d9-b813-317a46c0cc88', '307cc5c5-7479-4cf3-8e80-0f0468557ab7', ORDERUNIT, 'b95e9804-ab45-4523-83d8-0a3374da6592')"/>
				</field>
			</xsl:if>
			<xsl:if test="QUANTITYOFCUINTU">
				<field>
					<!--Кол-во в упаковке-->
					<xsl:attribute name="name" select="'QUANTITYOFCUINTU'"/>
					<xsl:value-of select="QUANTITYOFCUINTU"/>
				</field>
			</xsl:if>
			<xsl:if test="ORDERPRICE">
				<field>
					<!--Цена в системе покупателя без НДС-->
					<xsl:attribute name="name" select="'ORDERPRICE'"/>
					<xsl:value-of select="ORDERPRICE"/>
				</field>
			</xsl:if>
			<xsl:if test="PRICEWITHVAT">
				<field>
					<!--Цена в системе покупателя с НДС-->
					<xsl:attribute name="name" select="'PRICEWITHVAT'"/>
					<xsl:value-of select="PRICEWITHVAT"/>
				</field>
			</xsl:if>
			<xsl:if test="CHARACTERISTIC/DESCRIPTION">
				<field>
					<!--Наименование товара-->
					<xsl:attribute name="name" select="'DESCRIPTION'"/>
					<xsl:value-of select="CHARACTERISTIC/DESCRIPTION"/>
				</field>
			</xsl:if>
		</fieldset>
	</xsl:template>
</xsl:stylesheet>


Conversion rule example for incoming documents
==============================================

.. code:: xml

<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:saxon="http://saxon.sf.net/" xmlns:wdExtensions="java:com.whitedoc.xslt.extensions.WdExtensions" exclude-result-prefixes="saxon wdExtensions">
	<xsl:output indent="yes"/>

	<xsl:template match="/">
		<xsl:apply-templates select="envelope/documents/document"/>
	</xsl:template>

	<xsl:template match="document">
		<ORDER>
			<DOCUMENTNAME>220</DOCUMENTNAME>
			<NUMBER>
				<xsl:value-of select="field[@name='NUMBER']"/>
			</NUMBER>
			<DATE>
				<xsl:value-of select="field[@name='DATE']"/>
			</DATE>
			<DELIVERYDATE>
				<xsl:value-of select="field[@name='DELIVERYDATE']"/>
			</DELIVERYDATE>
			<CAMPAIGNNUMBER>
				<xsl:value-of select="field[@name='CAMPAIGNNUMBER']"/>
			</CAMPAIGNNUMBER>
			<xsl:if test="field[@name='CURRENCY']">
				<CURRENCY>
					<xsl:value-of select="field[@name='CURRENCY']"/>
				</CURRENCY>
			</xsl:if>
			<xsl:if test="field[@name='INFO']">
				<INFO>
					<xsl:value-of select="field[@name='INFO']"/>
				</INFO>
			</xsl:if>
			<xsl:if test="field[@name='DOCTYPE']">
				<DOCTYPE>
					<xsl:choose>
						<xsl:when test="field[@name='DOCTYPE']='Оригінал замовлення'">O</xsl:when>
						<xsl:when test="field[@name='DOCTYPE']='Заміна замовлення'">R</xsl:when>
						<xsl:when test="field[@name='DOCTYPE']='Видалення замовлення'">D</xsl:when>
						<xsl:when test="field[@name='DOCTYPE']='Фіктивність замовлення'">F</xsl:when>
						<xsl:when test="field[@name='DOCTYPE']='Попереднє замовлення'">PO</xsl:when>
						<xsl:when test="field[@name='DOCTYPE']='Замовлення на послугу / маркетинг'">OS</xsl:when>
						<xsl:otherwise>
							<xsl:value-of select="field[@name='DOCTYPE']"/>
						</xsl:otherwise>
					</xsl:choose>
				</DOCTYPE>
			</xsl:if>
			<HEAD>
				<SUPPLIER>
					<xsl:value-of select="field[@name='SUPPLIER']"/>
				</SUPPLIER>
				<BUYER>
					<xsl:value-of select="field[@name='BUYER']"/>
				</BUYER>
				<DELIVERYPLACE>
					<xsl:value-of select="field[@name='DELIVERYPLACE']"/>
				</DELIVERYPLACE>
				<SENDER>
					<xsl:value-of select="field[@name='SENDER']"/>
				</SENDER>
				<RECIPIENT>
					<xsl:value-of select="field[@name='RECIPIENT']"/>
				</RECIPIENT>
				<EDIINTERCHANGEID>
					<xsl:value-of select="field[@name='EDIINTERCHANGEID']"/>
				</EDIINTERCHANGEID>
				<xsl:apply-templates select="fieldgroup/fieldset"/>
			</HEAD>
		</ORDER>
	</xsl:template>

	<xsl:template match="fieldset">
		<POSITION>
			<POSITIONNUMBER>
				<xsl:value-of select="field[@name='POSITIONNUMBER']"/>
			</POSITIONNUMBER>
			<PRODUCT>
				<xsl:value-of select="field[@name='PRODUCT']"/>
			</PRODUCT>
			<PRODUCTIDSUPPLIER>
				<xsl:value-of select="field[@name='PRODUCTIDSUPPLIER']"/>
			</PRODUCTIDSUPPLIER>
			<PRODUCTIDBUYER>
				<xsl:value-of select="field[@name='PRODUCTIDBUYER']"/>
			</PRODUCTIDBUYER>
			<ORDEREDQUANTITY>
				<xsl:value-of select="field[@name='ORDEREDQUANTITY']"/>
			</ORDEREDQUANTITY>
			<ORDERPRICE>
				<xsl:value-of select="field[@name='ORDERPRICE']"/>
			</ORDERPRICE>
			<PRICEWITHVAT>
				<xsl:value-of select="field[@name='PRICEWITHVAT']"/>
			</PRICEWITHVAT>
			<ORDERUNIT>
				<xsl:variable name="ordunt" select="field[@name='ORDERUNIT']"/>
				<xsl:value-of select="wdExtensions:getValueFromDictionary('165dbb79-6ffa-40d9-b813-317a46c0cc88', 'b95e9804-ab45-4523-83d8-0a3374da6592', $ordunt, '307cc5c5-7479-4cf3-8e80-0f0468557ab7')"/>
			</ORDERUNIT>
			<CHARACTERISTIC>
				<DESCRIPTION>
					<xsl:value-of select="field[@name='DESCRIPTION']"/>
				</DESCRIPTION>
			</CHARACTERISTIC>
		</POSITION>
	</xsl:template>
</xsl:stylesheet>
