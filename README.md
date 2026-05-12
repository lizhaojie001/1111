@startuml
skinparam backgroundColor #FEFEFE
skinparam defaultTextAlignment center

start

:updatePublic(enableUpdate);

if (自动更新开启\n且 (enableUpdate 或 无 index.html)) then (否)
  stop
endif

fork
  :后台线程;
  :getUpdateReadySign;
  :startUpdate;
fork again
end fork

if (startUpdateReplace(false)\n且 TempSettings.UpdateReady?) then (是)
  :组命令行 (-u 等)\n灰度时 appendGrayArgs (-g -v);
  :beginUpdatePro → 启动 IteachUpdate;
  :sigUpdateFinish;
  stop
endif

:startDownloadUpdateApp;

if (getEnvironment / Navigator?) then (失败)
  stop
endif

:getLocalVersion;
:getCloudPublicVersion\n(Utils::getCloudVersion\n→ 升级点 POST 全国策略);

if (非灰度 且\nneedUpdatePublic 为 false?) then (是)
  stop
endif

if (initUpgradePlugin?) then (失败)
  stop
endif

:getUpgradeUrl\n(全国: update_client CDN);

:ITeachUpgradePlugin::startDownload;

:等待 sigFinish;

:组参数 (-d 等，可选 -u、-h);
:appendGrayArgs (仅灰度);

:beginUpdatePro → IteachUpdate;

:sigUpdateFinish;

stop
@enduml
