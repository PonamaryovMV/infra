# infra — Argo CD стенда

Argo CD **v3.6.0-rc4**, установленный из upstream-манифеста через kustomize и управляющий сам собой.
Первичную установку делает [`scripts/up.sh`](https://github.com/PonamaryovMV/scripts), дальше всё меняется коммитами сюда.

```
argocd/
├── kustomization.yaml        # install.yaml нужной версии + патчи + всё ниже
├── argocd-self.yaml          # Application "argocd" → эта папка (self-management)
├── projects.yaml             # AppProject: platform, workloads, kargo
├── repositories.yaml         # OCI-репозиторий чартов Kargo (enableOCI)
├── appsets/
│   ├── platform.yaml         # gitops-infra/components/*/component.yaml → Helm-чарт + values
│   ├── workloads.yaml        # gitops-main/apps/*/envs/*              → <app>-<env> в ns <env>
│   └── kargo-pipelines.yaml  # gitops-main/apps/*/kargo               → kargo-<app>
└── patches/
    ├── argocd-cmd-params-cm.yaml          # server.insecure (UI по http)
    ├── argocd-cm.yaml                     # url, опрос git раз в 30с
    ├── argocd-server-service.yaml         # NodePort 30080 → http://localhost:8080
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
- Аннотация `kargo.akuity.io/authorized-stage: <app>:<env>` на Application из `workloads` разрешает
  соответствующему Kargo-стейджу синкать его (`argocd-update`).
