﻿Telemetry, what is it about?
<h1>Telemetry explained</h1>
<p>Microsoft has quite a bit of information here about its new telemetry data system for SCCM here: <a href="https://technet.microsoft.com/en-US/library/mt613113.aspx">https://technet.microsoft.com/en-US/library/mt613113.aspx</a>
	</p>
<p>Below are my findings and additions to that documentation that people are inquiring about, but let me start off with the why, of it all. The new Configuration Manager comes with a brand new servicing mechanism. You should be aware by now that Windows 10 comes with a pretty high release cadence (a new Windows every 4 months). To keep up with that pace, Configuration Manager is planned to follow suit, and more or less follow that same cadence. Now, quite some people are sceptic about that increased cadence and the impact on the different products quality. To answer the challenges that come with this increased pace Microsoft plans to ship fast / fix fast, and that's were telemetry comes in.</p>
<p>The general idea is to find the setups and "modus operandi" that are frequently used by a large set of customers and use those in testing. Additionally, the telemetry data should prove to be a shortcut to get troubleshooting data to the product team. In other words, sharing data on the way you use the product should be in your own interest.</p>
<p>Sounds good, but am I supposed to just trust Microsoft in collecting data from my environment that might be privacy sensitive? Well, yes and no. Microsoft takes privacy extremely serious, and so does the Configuration Manager product team. The product team has, imho, very good reasons to make sure the data they collect isn't catalogued as privacy sensitive, as doing so would introduce them to a drastically increased involvement of both legal and auditing, as for any Microsoft service that holds privacy sensitive data. To keep the level of scrutiny they'd have to go through in check, the ConfigMgr team starts off with anonymizing the data. They do that by hashing some of the data, so that any privacy sensitive data isn't readable to them.</p>
<p>And that's where some of the challenges come in for customers that are privacy sensitive and/or have auditing breath down their neck. This hashed data isn't readable to them either, which worries some of them, as they don't know what they are sending out. Below I'll explain, for all the hashed values I found in telemetry results so far how you can make the data human readable alongside the hash so that you can control what the data you are sending out actually means.</p>
<h1>Database objects involved<br />
</h1>
<h2>Tables<br />
</h2>
<p>The <strong>telemetry table</strong> contains the names and id's of the stored procedures that are responsible for collecting the telemetry data. In the environments that I've verified this on, there are 150 stored procedures lists in  the telemetry table. You can have a look at this info by running the following query</p>
<style="margin-left: 36pt">
<style="margin-left: 36pt"><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">select</span>
			<span style="color:gray">*</span>
			<span style="color:blue">from</span> Telemetry 
            <span style="color:blue">order</span>
			<span style="color:blue">by</span> name
</span>
<style="margin-left: 36pt">
<span style="font-family:Consolas; font-size:9pt"><br />
		</span>
<p>The results are stored in a table called  TEL_telemetryresults. You can look at your own results by running the following query</p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">select</span>
			<span style="color:gray">* </span><span style="color:blue">from</span> TEL_TelemetryResults<br />
</span></p>
<span style="font-style:normal;" depending on your environment.</span><br />
<p>Depending upon the level of data you've chosen you should see a number of rows returned. There should be one thing that catches your eye quite swiftly. As you can see in the screenshot below each row has a results column that ends with a returning hash. Which opens up the very first question, what is this hash all about?<br />
</p>
<p><span style="font-style:normal;">Well this particular hash is used to correlate data between the different rows in your telemetry results so the product team can store all data coming from one customer together. Given the introduction they need a way to do that without making your company name or anything similar that could identify your environment, and hence they need to anonymize the data. Now, every Configuration Manager environment has a randomly generated hierarchyid that could be used for this purpose. But even that wasn't anoynymous enough for the Configuration Manager product team. To anonymize the data they've chosen to hash that hierarchy id using <strong>SHA256.<br />
</strong></span></p>
<a href="http://i.imgur.com/PakPsmw.png"><img src="http://i.imgur.com/PakPsmw.png"></a>


<p><span>You can get your own hierarchy id and the accompanying hash to validate this data by running the following query:<br />
</span></p>
<span style="font-family:Consolas; font-size:9pt">
<span style="color:blue">Declare</span> @tenantid <span style="color:blue">as</span>
			<span style="color:blue">nvarchar<span style="color:gray">(<span style="color:fuchsia">max<span style="color:gray">)</span>
					</span></span></span></span></p>
<span style="font-family:Consolas; font-size:9pt"><span style="color:blue">select</span> @TenantId <span style="color:gray">=</span> dbo<span style="color:gray">.</span>fnConvertBinaryToBase64String<span style="color:gray">(</span>dbo<span style="color:gray">.</span>fnMDMCalculateHash<span style="color:gray">(<span style="color:fuchsia">CONVERT<span style="color:gray">(<span style="color:blue">VARBINARY<span style="color:gray">(<span style="color:fuchsia">MAX<span style="color:gray">),</span> [dbo]<span style="color:gray">.</span>[</span>fnGetHierarchyID]()),</span>
							<span style="color:red">'SHA256'<span style="color:gray">)</span>
								<span style="color:gray">)</span><br />
							</span></span></span></span></span></span></p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">Declare</span> @hierarchyid <span style="color:blue">as</span>
			<span style="color:blue">nvarchar<span style="color:gray">(<span style="color:fuchsia">max<span style="color:gray">)</span><br />
					</span></span></span></span></p>
<p><span style="color:blue">select</span> @hierarchyid <span style="color:gray">=</span> [dbo]<span style="color:gray">.</span>[fnGetHierarchyID]<span style="color:gray">()</span><br />
		</span></p>
<p><span style="color:blue">select</span> @hierarchyid<span style="color:gray">,</span> @tenantid<br />
</span></span></p>
<p>
 </p>
<h3>Stored procedures<br />
</h3>
<p>There are a bunch of stored procedures involved in collecting the telemetry data, and most of them just generate just 1 of the rows in the telemetryresults table. You can find the stored procedures responsible for collecting data by running the following query.</p>
<p>
 </p>
<p>    <span style="font-family:Consolas; font-size:9pt"><span style="color:blue">SELECT</span><br />
			<span style="color:blue">distinct</span> o<span style="color:gray">.</span>name <span style="color:blue">As</span><br />
			<span style="color:red">'Stored Procedures'<span style="color:gray">,</span>o<span style="color:gray">.*</span><br />
			</span></span></p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">FROM</span><br />
			<span style="color:green">SYSOBJECTS</span> o <span style="color:gray">INNER</span><br />
			<span style="color:gray">JOIN</span><br />
			<span style="color:green">SYSCOMMENTS</span> c<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">ON</span> o<span style="color:gray">.</span>id <span style="color:gray">=</span> c<span style="color:gray">.</span>id<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">WHERE</span> o<span style="color:gray">.</span>name <span style="color:gray">like</span><br />
			<span style="color:red">'tel_%'</span><br />
			<span style="color:gray">and </span>o<span style="color:gray">.</span>xtype <span style="color:gray">=</span><br />
			<span style="color:red">'P'<br />
</span></span></p>
<p>
 </p>
<p>If you're only interested in the ones that generate data for the <strong>telemetryresults</strong> table run the query below.</p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">SELECT</span><br />
			<span style="color:blue">distinct</span> o<span style="color:gray">.</span>name <span style="color:blue">As</span><br />
			<span style="color:red">'Stored Procedures'<span style="color:gray">,</span>o<span style="color:gray">.*</span><br />
			</span></span></p>
<p style="margin-left: 36pt"><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">FROM</span><br />
			<span style="color:green">SYSOBJECTS</span> o <span style="color:gray">INNER</span><br />
			<span style="color:gray">JOIN</span><br />
			<span style="color:green">SYSCOMMENTS</span> c<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">ON</span> o<span style="color:gray">.</span>id <span style="color:gray">=</span> c<span style="color:gray">.</span>id<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">WHERE</span> o<span style="color:gray">.</span>name <span style="color:gray">like</span><br />
			<span style="color:red">'tel_%'</span><br />
			<span style="color:gray">and </span>o<span style="color:gray">.</span>xtype <span style="color:gray">=</span><br />
			<span style="color:red">'P' <span style="color:gray">and</span> o<span style="color:gray">.</span>name <span style="color:gray">in<span style="color:blue"><br />
						<span style="color:gray">(<span style="color:blue">select</span> name <span style="color:blue">from</span> Telemetry)<br />
</span></span></span></span></span></p>
<p>
 </p>
<p>You could subsequently analyze the stored procedures to see what it is they are collecting, but that is an elaborate exercise. As we've seen that SHA256 is the hashing mechanism of choice I've chosen to check which of these stored procedures use the SHA256 function. I've identified the stored procedures, and linked id's using this query</p>
<span style="font-family:Consolas; font-size:9pt"><span style="color:blue">SELECT</span>			<span style="color:blue">DISTINCT</span>       o<span style="color:gray">.</span>name <span style="color:blue">AS</span><span style="color:fuchsia"> Object_Name<span style="color:gray">,</span> Telemetry<span style="color:gray">.</span>id<span style="color:gray">,</span></span></span><span style="font-family:Consolas; font-size:9pt">       o<span style="color:gray">.</span>type_desc<span style="color:gray">,</span> m<span style="color:gray">.<span style="color:blue">definition</span></span></span> 
<span style="font-family:Consolas; font-size:9pt"><span style="color:blue">FROM </span><span style="color:green">sys<span style="color:gray">.<span style="color:green">sql_modules</span> m </span></span></span><span style="font-family:Consolas; font-size:9pt">
<span style="color:gray">INNER</span><span style="color:gray">JOIN</span><span style="color:green">sys<span style="color:gray">.<span style="color:green">objects</span> o <span style="color:blue">ON</span> m.<span style="color:fuchsia">object_id</span> =</span> o<span style="color:gray">.<span style="color:fuchsia">object_id</span>
				</span></span></span>
<p><span style="font-family:Consolas; font-size:9pt">    <span style="color:gray">inner</span><br />
			<span style="color:gray">join</span> telemetry <span style="color:blue">on</span> o<span style="color:gray">.</span>name <span style="color:gray">=</span> Telemetry<span style="color:gray">.</span>Name<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt">    <span style="color:blue">where</span> m<span style="color:gray">.<span style="color:blue">definition</span> like</span><br />
			<span style="color:red">'%sha256%'</span><br />
			<span style="color:gray">and</span> o<span style="color:gray">.</span>name <span style="color:gray">like</span><br />
			<span style="color:red">'tel_%'<br />
</span></span></p>
<p>
 </p>
<p>This results in the following list of id's</p></p>
<p>
 </p>
<p>Which in turn lets you focus on the <strong>telemeteryresults </strong>table and the rows that contain hashed information:</p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">select</span><br />
			<span style="color:gray">*</span><br />
			<span style="color:blue">from</span> TEL_TelemetryResults<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:blue">where</span> id <span style="color:gray">in</span><br />
		</span></p>
<p><span style="color:gray; font-family:Consolas; font-size:9pt">(<span style="color:red">'ACABF386-BCD1-48C5-9C7F-A33DADA6E89D'<span style="color:gray">,</span><br />
				<span style="color:green">--TEL_Content_DPState</span><br />
			</span></span></p>
<p><span style="color:red; font-family:Consolas; font-size:9pt">'69FC4B89-3561-4360-9157-4F8E896F7FB9'<span style="color:gray">,</span><br />
			<span style="color:green">--TEL_Content_Package</span><br />
		</span></p>
<p><span style="color:red; font-family:Consolas; font-size:9pt">'2E8CC4FA-738D-4A48-B36F-E981344C97C3'<span style="color:gray">,</span><br />
			<span style="color:green">--TEL_DCM_BuiltinSettings</span><br />
		</span></p>
<p><span style="color:red; font-family:Consolas; font-size:9pt">'942B1F7E-EB3F-4576-8CB8-F8066D31940F'<span style="color:gray">,</span><br />
			<span style="color:green">--TEL_EAS_Connectors</span><br />
		</span></p>
<p><span style="color:red; font-family:Consolas; font-size:9pt">'CD6B1D69-5F70-46B1-BC82-2C99764188B5'<span style="color:gray">,</span><br />
			<span style="color:green">--TEL_MAM_PolicySettingStatistics4Deployment2Collection</span><br />
		</span></p>
<p><span style="color:red; font-family:Consolas; font-size:9pt">'3B694B4A-DA65-4E60-BAE9-5796849A9586'<span style="color:gray">,</span><br />
			<span style="color:green">--TEL_Perf_TableSize</span><br />
		</span></p>
<p><span style="color:red; font-family:Consolas; font-size:9pt">'E1201168-0A70-41B7-857E-309F8A5FB96B'<span style="color:gray">,</span><br />
			<span style="color:green">--TEL_SetupInfo</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><span style="color:red">'0F40B971-AAC7-4A39-8CDA-1E023C833306'</span><br />
			<span style="color:gray">)</span><br />
			<span style="color:green">--TEL_SQL_DBSchema<br />
</span></span></p>
<p>
 </p>
<p>Or on those that should not contain any hashed information by changing the where clause to use not in instead of in.  This should allow you to quickly check whether the results column still has data you can't understand. (Should that be the case feel free to share the ID of the row and I'll happily look into it.)</p>
<p>
 </p>
<h1>Obfuscated data / data hashing and making it human readable again<br />
</h1>
<p>The last ID <span style="color:red; font-family:Consolas; font-size:9pt">'0F40B971-AAC7-4A39-8CDA-1E023C833306' </span>contains the full schema of your Configuration Manager database as collected by the <strong>TEL_SQL_DBSCHEMA</strong> stored procedure. When you look at the stored procedure definition you'll notice that it runs the following query to collect the data:</p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue; background-color:yellow">SELECT</span><span style="background-color:yellow"> dbo<span style="color:gray">.</span>fnConvertBinaryToBase64String<span style="color:gray">(</span></span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="background-color:yellow">dbo<span style="color:gray">.</span>fnMDMCalculateHash<span style="color:gray">(<span style="color:fuchsia">CONVERT<span style="color:gray">(<span style="color:blue">VARBINARY<span style="color:gray">(<span style="color:fuchsia">MAX<span style="color:gray">),</span> DS<span style="color:gray">.</span>ObjectName<span style="color:gray">),</span><br />
										<span style="color:red">'SHA256'<span style="color:gray">))</span><br />
											<span style="color:blue">AS</span> ObjectNameHash<span style="color:gray">,</span></span></span></span></span></span></span></span></span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>ObjectVersion <span style="color:blue">AS</span> ObjectVersion<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>UpdatedBy <span style="color:blue">AS</span> UpdatedBy<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>ObjectHash <span style="color:blue">As</span> ObjectHash<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">FROM</span>   dbo<span style="color:gray">.</span>DBSchema DS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:gray">INNER</span><br />
			<span style="color:gray">JOIN</span> SC_SiteDefinition SS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">ON</span> DS<span style="color:gray">.</span>SiteNumber <span style="color:gray">=</span> SS<span style="color:gray">.</span>SiteNumber<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">WHERE</span><br />
			<span style="color:fuchsia">ISNULL<span style="color:gray">(</span>SS<span style="color:gray">.</span>parentsitecode<span style="color:gray">,</span><br />
				<span style="color:red">N''<span style="color:gray">)</span><br />
					<span style="color:gray">=</span> N''<br />
</span></span></span></p>
<p>
 </p>
<p>As should be apparent, the objectnames are obfuscated in this stored procedure. Should you like to know what the obfuscated data really means you can modify the query slightly and another item in the select section of the query to include the data before it is hashed like so:</p>
<p>
 </p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue; background-color:yellow">SELECT</span><span style="background-color:yellow"> DS<span style="color:gray">.</span>ObjectName<span style="color:gray">,</span> dbo<span style="color:gray">.</span>fnConvertBinaryToBase64String<span style="color:gray">(</span></span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="background-color:yellow">dbo<span style="color:gray">.</span>fnMDMCalculateHash<span style="color:gray">(<span style="color:fuchsia">CONVERT<span style="color:gray">(<span style="color:blue">VARBINARY<span style="color:gray">(<span style="color:fuchsia">MAX<span style="color:gray">),</span> DS<span style="color:gray">.</span>ObjectName<span style="color:gray">),</span><br />
										<span style="color:red">'SHA256'<span style="color:gray">))</span><br />
											<span style="color:blue">AS</span> ObjectNameHash<span style="color:gray">,</span></span></span></span></span></span></span></span></span><strong><br />
			</strong></span></p>
<p><span style="font-family:Consolas; font-size:9pt">      DS<span style="color:gray">.</span>ObjectVersion <span style="color:blue">AS</span> ObjectVersion<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">        DS<span style="color:gray">.</span>UpdatedBy <span style="color:blue">AS</span> UpdatedBy<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>ObjectHash <span style="color:blue">As</span> ObjectHash<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">FROM</span>   dbo<span style="color:gray">.</span>DBSchema DS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:gray">INNER</span><br />
			<span style="color:gray">JOIN</span> SC_SiteDefinition SS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">ON</span> DS<span style="color:gray">.</span>SiteNumber <span style="color:gray">=</span> SS<span style="color:gray">.</span>SiteNumber<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">WHERE</span><br />
			<span style="color:fuchsia">ISNULL<span style="color:gray">(</span>SS<span style="color:gray">.</span>parentsitecode<span style="color:gray">,</span><br />
				<span style="color:red">N''<span style="color:gray">)</span><br />
					<span style="color:gray">=</span> N''</span><br />
			</span><br />
		</span></p>
<p>
 </p>
<p>As you can see, all I did was include the column DS.ObjectName before it was hashed so you could see it in readable format alongside the hashed format. The reason they hash the data in this particular instance is because your're schema could contain your company name, or other privacy sensitive data. The most likely way this would end up in your schema is by including that information in the names of your custom hardware inventory classes.</p>
<p>This is just one of the 8 queries that might contain hashed data, but the mechanism above is repeatable for the other stored procedures. I'll add the queries needed to represent the cleartext data and the hashed variant over the next couple of days.</p>
<p><span style="color:#555555; font-family:Arial; font-size:9pt">Enjoy.<br />"The M in WMI stands for Magic"<br />""Everyone is an expert at something" Kim Oppalfens - ConfigMgr Expert for lack of any other expertise<br />System Center Configuration Manager MVP<br />http://www.scug.be/thewmiguy/default.aspx</p>
<p>http://www.linkedin.com/in/kimoppalfens</p>
<p>http://twitter.com/thewmiguy</span></p>