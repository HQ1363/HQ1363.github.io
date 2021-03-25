<ul>
<li>对于错误计数，使用counter类型，即代码里使用<code>increment</code>方法</li>
<li>对于调用频次、调用时间，使用timer类型，即代码里使用<code>timing</code>方法</li>
<li>对于系统load等已有指标，发送gauge</li>
</ul></li>
<li>指标命名中<strong>不应该包含机器名、集群名以及参数变量</strong></li>
</ol>
<h2><a id="Graphite/Grafana指标命名含义" class="anchor" href="#Graphite/Grafana指标命名含义">
               <span class="octicon octicon-link"></span></a>
               Graphite/Grafana指标命名含义</h2>
<ul>
<li><code>stats_counts.*</code>                10秒内总的计数(单位:个/10秒)</li>
<li><code>stats.*</code>                       每秒的计数(单位:个/秒)</li>
<li><code>stats.timers.*.count_ps</code>        每秒的调用次数(单位:次/秒)</li>
<li><code>stats.timers.*.mean_90</code>         每次调用的时长(去掉合计周期内最慢10%后的平均值, 单位: 毫秒)</li>
<li><code>stats.timers.*.upper_90</code>        每次调用的时长(去掉合计周期内最慢10%后的最大值, 单位: 毫秒)</li>
</ul>

<p>对于timers的后缀:</p>

<ul>
<li><code>count</code>   取样区间(10s)内计数</li>
<li><code>count_{90,95}</code>   去掉最慢 {10%,5%} 的计数</li>
<li><code>lower</code>       最短时长</li>
<li><code>mean</code>    平均时长</li>
<li><code>mean_{90,95}</code> 去掉最慢 {10%,5%} 的时长</li>
<li><code>median</code> 时长中位数</li>
<li><code>std</code> 时长标准差</li>
<li><code>sum</code> 总时长</li>
<li><code>sum_{90,95}</code> 去掉最慢 {10%,5%} 的总时长</li>
<li><code>sum_suqares</code> 时长平方和</li>
<li><code>upper</code> 时长最大值</li>
<li><code>upper_{90,95}</code> 去掉最慢 {10%,5%} 的时长</li>
</ul>
<h2><a id="Bell指标命名含义" class="anchor" href="#Bell指标命名含义">
               <span class="octicon octicon-link"></span></a>
               Bell指标命名含义</h2>
<p>与Graphite的命名策略不同，bell采取的是前缀命名方式。</p>

<ul>
<li><code>timer.mean_90.*</code>     每次调用的响应时间(单位:毫秒)</li>
<li><code>timer.count_ps.*</code>    每秒的调用次数(单位:次/秒)</li>
<li><code>counter.*</code>           每秒的计数(单位:个/秒)</li>
</ul>
<h2><a id="Bell报警规则添加建议(for noc)" class="anchor" href="#Bell报警规则添加建议(for noc)">
               <span class="octicon octicon-link"></span></a>
               Bell报警规则添加建议(for noc)</h2>
<ul>
<li>时长类指标(<code>timer.mean_90.*</code>): 只关心上升趋势，不关心下降趋势，并且最好设指标的值下限</li>
<li>调用次数类指标(<code>timer.count_ps.*</code>): 上升和下降趋势都要关心，并且最好设好指标的值下限</li>
</ul>
