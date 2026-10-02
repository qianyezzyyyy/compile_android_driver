obj-m += lsdriver.o
lsdriver-y := lsdriver_main.o \
			  arm64_encode/arm64_encode.o \
			  arm64_decode/arm64_decode.o \
			  arm64_decode/arm64_decode_immediate.o \
			  arm64_decode/arm64_decode_register.o \
			  arm64_decode/arm64_decode_ldst.o \
			  arm64_decode/arm64_decode_branch.o \
			  arm64_decode/arm64_decode_simd.o \
			  arm64_decode/arm64_decode_sve.o \
			  arm64_decode/arm64_decode_sme.o \
			  arm64_emulate/emulate_inst.o \
			  arm64_emulate/arm64_emulate_branch.o \
			  arm64_emulate/arm64_emulate_ldst.o \
			  arm64_emulate/arm64_emulate_simd.o \
			  arm64_emulate/arm64_emulate_immediate.o \
			  arm64_emulate/arm64_emulate_register.o

$(obj)/lsdriver_main.o: $(src)/lsdriver.c FORCE
	$(call if_changed_dep,cc_o_c)

# O3 优化
ccflags-y += -O3

# 5.10 上先不启用模块级 full LTO，避免与内核 LTO=thin / 重定位冲突。
# 构建跑通、真机验证稳定后再评估是否开启。
# ccflags-y += -flto

# 开启 debug 输出；需要 pr_debug 时手动启用。
# ccflags-y += -DDEBUG

# 关键改动：Android 12 GKI 5.10 用的 Clang(12/14) 对 +bf16+i8mm 支持不完整。
# 降级到 armv8.5-a 稳妥组合；更新的指令继续在模板里用 .inst 机器码。
ccflags-y += -march=armv8.5-a+fp16+fp16fml+lse+rcpc+crc

# 禁止优化器从普通 C 循环和基本块主动生成 Advanced SIMD 向量代码。
ccflags-y += -fno-vectorize -fno-slp-vectorize

# 仅从显式使用 FP/SIMD 汇编的目标中移除 -mgeneral-regs-only
CFLAGS_REMOVE_lsdriver_main.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate/emulate_inst.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate/arm64_emulate_ldst.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate/arm64_emulate_simd.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate/arm64_emulate_immediate.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate/arm64_emulate_register.o += -mgeneral-regs-only

CFLAGS_lsdriver_main.o += -mno-implicit-float
CFLAGS_arm64_emulate/emulate_inst.o += -mno-implicit-float
CFLAGS_arm64_emulate/arm64_emulate_ldst.o += -mno-implicit-float
CFLAGS_arm64_emulate/arm64_emulate_simd.o += -mno-implicit-float
CFLAGS_arm64_emulate/arm64_emulate_immediate.o += -mno-implicit-float
CFLAGS_arm64_emulate/arm64_emulate_register.o += -mno-implicit-float

# 将所有警告降级，不再视为错误。
ccflags-y += -Wno-error

# 使用现代 GNU C 方言。
ccflags-y += -std=gnu11

# 关闭栈保护器。
ccflags-y += -fno-stack-protector

# 省略帧指针。
ccflags-y += -fomit-frame-pointer

# 尾调用优化。
ccflags-y += -foptimize-sibling-calls

# 关键改动：Android 12 GKI 5.10 开启了 BTI/PAC，
# 绝不能加 -mbranch-protection=none。
# 原注释里描述的 CFI jump table / BTI panic 是 6.1 + CFI 的坑，5.10 上没有 KCFI。
# Kbuild 会按内核 config 自动带上正确的 branch protection，无需手动指定。
#ccflags-y += -mbranch-protection=none

# 关闭自动局部变量清零。
ccflags-y += -ftrivial-auto-var-init=uninitialized

# 循环展开。
ccflags-y += -funroll-loops

# 严格别名优化。
ccflags-y += -fstrict-aliasing

# 关闭各类 sanitizer 插桩
KASAN_SANITIZE := n
UBSAN_SANITIZE := n
KCSAN_SANITIZE := n
KMSAN_SANITIZE := n
GCOV_PROFILE := n
KCOV_INSTRUMENT := n

# 放宽栈帧上限（针对本模块，避免全树 sed）
ccflags-y += -Wframe-larger-than=4096

# ---- 以下为保留的“不要启用”项，仅作注释留档 ----
# ccflags-y += -fdelete-null-pointer-checks
# ccflags-y += -fstrict-overflow
# ccflags-y += -finline-functions
# ccflags-y += -mllvm -inline-threshold=10000
# ccflags-y += -mllvm -unroll-threshold=10000
# ccflags-y += -fvisibility=hidden
# ccflags-y += -fno-plt
# ccflags-y += -fno-jump-tables
# ccflags-y += -fno-sanitize=shadow-call-stack
# ccflags-y += -fno-sanitize=kcfi
# CFLAGS_REMOVE_lsdriver.o += $(CC_FLAGS_FTRACE)
# ccflags-y += -mtune=cortex-x2
