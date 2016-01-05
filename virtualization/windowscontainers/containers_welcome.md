# Windows コンテナーに関するドキュメント

Windows コンテナーは、1 つのシステムで複数の独立したアプリケーションを実行できるようにする、オペレーティング システム レベルの仮想化を実現します。 この機能では、2 種類のコンテナー ランタイムを使用します。それぞれ、アプリケーションの分離の度合いが異なります。 Windows Server コンテナーは、名前空間とプロセスの分離によって分離を実現します。 Hyper-V コンテナーは、軽量の仮想マシンで各コンテナーをカプセル化します。 2 種類のランタイムに加えて、この両方を PowerShell または Docker で管理できます。 このドキュメント セットでは、クイック スタート ガイド、展開ガイド、管理操作に関する詳しい技術情報を提供します。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:80%" cellpadding="25" cellspacing="5">
<tr>
<td><center>![](media/try.png)</center></td>
<td>**クイック スタート**<br /><br />
Windows Server コンテナーと Hyper-V コンテナーを試す場合は、次のクイック スタート ガイドをご利用ください。<br /><br />
<ul>
<li>[Azure クイック スタート](quick_start/azure_setup.md)<br /><br /></li>
<li>[新しいコンテナー ホストの展開](quick_start/container_setup.md)<br /><br /></li>
<li>[既存のシステムでのホストの展開](quick_start/inplace_setup.md)<br /><br /></li>
<li>[PowerShell クイック スタート](quick_start/manage_powershell.md)<br /><br /></li>
<li>[Docker クイック スタート](quick_start/manage_docker.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td><center>![](media/1.png)</center></td>
<td>**展開**<br /><br />
Windows Server 2016 と Nano Server で Windows コンテナーを展開する方法について確認できます。<br /><br />
<ul>
<li>[システム要件](deployment/system_requirements.md)<br /><br /></li>
<li>[コンテナー ホストの展開](deployment/deployment.md)<br /><br /></li>
<li>[Windows での Docker の展開](deployment/docker_windows.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td><center>![](media/explore.png)</center></td>
<td>**管理**<br /><br />
Windows Server 2016 と Nano Server での Windows コンテナーの管理について確認できます。<br /><br />
<ul>
<li>[コンテナーの管理](management/manage_containers.md)<br /><br /></li>
<li>[イメージの管理](management/manage_images.md)<br /><br /></li>
<li>[ネットワークの管理](management/container_networking.md)<br /><br /></li>
<li>[コンテナー データの管理](management/manage_data.md)<br /><br /></li>
<li>[Hyper-V コンテナーの管理](management/hyperv_container.md)<br /><br /></li>
<li>[コンテナー リソースの管理](management/manage_resources.md)<br /><br /></li>
<li>[コンテナーの相互運用性](management/hcs_powershell.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td><center>![](media/question.png)</center></td>
<td>**コミュニティ**<br /><br />
コミュニティでやり取りしたり、サンプルを試してみたりできるほか、その他のリソースを探すこともできます。<br /><br />
<ul>
<li>[コンテナーのフォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)<br /><br /></li>
<li>[コミュニティ リソース](https://msdn.microsoft.com/virtualization/community/community_overview)<br /><br /></li>
<li>[サンプルとスクリプト](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-samples)<br /><br /></li>
</ul>
</td>
</tr>
</table>




