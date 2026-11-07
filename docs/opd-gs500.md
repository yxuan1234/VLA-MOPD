# Released Student: global step 500 / 发布的蒸馏模型

The released distilled Student is:

**[yxuan1234/Pi05-LIBERO-OPD-Student-gs500](https://huggingface.co/yxuan1234/Pi05-LIBERO-OPD-Student-gs500)**

Revision: `282d6ce8dd9c1ce34ea2bfe06fb88a40cee52cdf`.
Weights: `actor/model_state_dict/full_weights.pt` (8,683,564,902 bytes).
SHA-256: `5ceb9a370f235224045450f73328250f3d06d5156ee0ae763fc60f1208565012`.
This is a full checkpoint with unmerged language-model-only LoRA, rank 32,
252 adapter tensors. It is not an adapter-only download.

发布的 500-step Student 权重包含完整模型和未合并的
language-model-only LoRA；评测入口恢复权重并在内存中 merge。

## Download / 下载

Install OpenPI + LIBERO following the official RLinf installation guide and
the repository README. Run these commands from the repository root:

```bash
hf download yxuan1234/Pi05-LIBERO-OPD-Student-gs500 \
  --revision 282d6ce8dd9c1ce34ea2bfe06fb88a40cee52cdf \
  --local-dir /path/to/Pi05-LIBERO-OPD-Student-gs500

export PI05_OPD_CHECKPOINT_DIR=/path/to/Pi05-LIBERO-OPD-Student-gs500/actor
export PI05_SFT_MODEL_DIR=/path/to/RLinf-Pi05-LIBERO-SFT
export EMBODIED_PATH="$PWD/examples/embodiment"
export ROBOT_PLATFORM=LIBERO
export LIBERO_TYPE=standard
export MUJOCO_GL=egl
export PYOPENGL_PLATFORM=egl
export OPD_GPU_PLACEMENT=0-1
export RAY_ADDRESS=127.0.0.1:YOUR_RAY_PORT
```

The base SFT model is required by the release loader. Connect to your allocated
Ray cluster before evaluation. `OPD_GPU_PLACEMENT` denotes physical GPU ranks.

需要另行准备 Base SFT，并连接已分配给自己的 Ray 集群。

## Evaluation configs and commands / 评测配置和启动命令

Configs live under `evaluations/libero/` and inherit
`_libero_opd_vlm_lora_eval.yaml`. Each suite config defines its own episode
horizon and rollout length. The shared config defaults to `flow_ode`; use
`flow_sde` with noise level 0.3 for the reported gs500 results:

```bash
for suite in spatial object goal 10; do
  python evaluations/eval_embodied_agent.py \
    --config-path libero \
    --config-name "libero_${suite}_opd_vlm_lora_eval" \
    env.eval.seed=0 \
    env.eval.use_ordered_reset_state_ids=false \
    rollout.model.openpi.noise_method=flow_sde \
    rollout.model.openpi.noise_level=0.3 \
    rollout.model.openpi.train_expert_only=true \
    runner.logger.log_path="/path/to/evaluation-results/gs500/$suite" || break
done
```

Evaluation settings: 50 concurrent environments, one rollout epoch, ten
episode horizons per rollout, 500 reported trajectories per suite, fixed reset
state IDs, auto-reset enabled, terminations ignored, 256×256 images, five
denoising steps and ten-action chunks. Goal uses horizon **320**;
LIBERO-10 uses **520**.

评测使用 `flow_sde`、noise level 0.3、50 个并行环境，每个 suite
评测 500 个 episode。上述命令设置相应参数，评测入口会在内存中合并 LoRA。

## Evaluation results / 评测结果

| Suite | success_once | Successful episodes | success_at_end |
| --- | ---: | ---: | ---: |
| Spatial | 99.4% | 497/500 | 97.6% |
| Object | 99.0% | 495/500 | 96.8% |
| Goal | 98.0% | 490/500 | 80.2% |
| LIBERO-10 | 99.0% | 495/500 | 82.4% |
| Mean | 98.85% | 1977/2000 | 89.25% |

Results are from the gs500 training-run evaluation logs. The release evaluation
commands have been checked for configuration composition but have not been rerun
end to end with the release loader. `success_once` and `success_at_end` are
separate metrics.

以上结果来自 gs500 训练运行的评测日志；发布版评测命令已完成配置组合检查，
尚未使用发布版加载器重新完成端到端评测。
