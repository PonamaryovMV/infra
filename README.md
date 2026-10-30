# infra — Argo CD стенда

Argo CD **v3.6.0-rc4**, установленный из upstream-манифеста через kustomize и управляющий сам собой.
Первичную установку делает [`scripts/windows/up.ps1`](https://github.com/PonamaryovMV/scripts), дальше всё меняется коммитами сюда.

```
argocd/
├── kustomization.yaml        # install.yaml нужной версии + патчи + всё ниже
├── argocd-self.yaml          # Application "argocd" → эта папка (self-management)
├── ingress.yaml              # http://argocd через Traefik
├── projects.yaml             # AppProject: platform, workloads, kargo
├── repositories.yaml         # OCI-репозитории чартов: Kargo и общий чарт main
├── appsets/
│   ├── platform.yaml         # gitops-infra/components/*/component.yaml → Helm-чарт + values
│   ├── workloads.yaml        # gitops-main/<АС>/<ФП>/<сервис>/<окр>  → <АС>-<ФП>-<сервис>-<окр> (Helm: values + release.yaml)
│   ├── workloads-prod.yaml   # gitops-prod/<АС>/<ФП>/<сервис>/prod  → <АС>-<ФП>-<сервис>-prod
│   └── kargo-pipelines.yaml  # gitops-main/kargo/<проект>           → kargo-<проект>
└── patches/
    ├── argocd-cmd-params-cm.yaml          # server.insecure (UI по http)
    ├── argocd-cm.yaml                     # url http://argocd, опрос git раз в 30с
    └── argocd-server-rollout-extension.yaml  # UI Argo Rollouts внутри Argo CD
```

## Обновить Argo CD

Поменяйте версию в URL `install.yaml` в `argocd/kustomization.yaml` и закоммитьте — Application `argocd` обновит сам себя.
При переходе с RC на GA сверьтесь с [заметками по апгрейду](https://argo-cd.readthedocs.io/en/latest/operator-manual/upgrading/overview/).

## Заметки

- Везде `ServerSideApply=true`: CRD Argo CD и Kargo не влезают в аннотацию client-side apply.
- У `argocd` выключен `prune` — чтобы ошибка в kustomization не снесла сам Argo CD.
- Между Application платформы нет порядка (Kargo нужны CRD cert-manager). Вместо sync-wave — бесконечный `retry`:
  Kargo просто досинкается, когда появятся CRD.
- Аннотация `kargo.akuity.io/authorized-stage: <АС>-<ФП>:<окружение>` на Application'ах из `workloads` и `workloads-prod`
  разрешает соответствующему Kargo-стейджу синкать их (`argocd-update`).
