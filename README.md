# gpu-deploy-triage-demo

GPU 워크로드의 **배포 실패 triage**를 닫힌 루프로 시연하는 데모. ArgoCD가 Degraded를 감지하면 Slack으로 알리고, 에이전트가 read-only로 진단해 근거와 다음 행동을 제시하며, 사람이 승인하면 가역 조치(롤백)까지 닫는다.

이 레포는 그 데모의 **배포 대상 앱**(Helm 차트)만 담는다 — ArgoCD가 이 repo를 pull한다.

## 구성

```
chart/gpu-sim/
├── Chart.yaml
├── values.yaml          # healthy v1 ↔ broken v2 (이미지 태그·parallelism)
└── templates/
    ├── deployment-api.yaml   # 항상 Healthy한 API
    └── job-sim.yaml          # GPU 시뮬레이션 Job (깨지는 워크로드)
```

## 두 가지 실패를 의도적으로 재현

| 상태 | 어떻게 | 진단 |
|---|---|---|
| `CrashLoopBackOff` | 컨테이너 CUDA 런타임(12.6) > 노드 드라이버(12.2 한계) | 롤백으로 해결 |
| `Pending` | GPU 1개뿐인데 parallelism 3 → `Insufficient nvidia.com/gpu` | 롤백으로 **안** 됨 → 스케줄링/캐파 |

GPU 시뮬레이션 이미지는 [cpp-build-optimization-reference](https://github.com/pghoya2956/cpp-build-optimization-reference)의 CUDA 빌드를 재사용한다.

## 배포

ArgoCD Application이 이 repo의 `chart/gpu-sim`을 `gpu-demo` 네임스페이스로 sync. 공개 repo라 별도 자격 없이 pull된다.
