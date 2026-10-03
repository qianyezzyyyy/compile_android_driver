obj-m += lsdriver.o

lsdriver-y := lsdriver_main.o \
              arm64_decode.o \
              arm64_decode_branch.o \
              arm64_decode_immediate.o \
              arm64_decode_ldst.o \
              arm64_decode_register.o \
              arm64_decode_simd.o \
              arm64_decode_sme.o \
              arm64_decode_sve.o \
              arm64_encode.o \
              emulate_inst.o \
              arm64_emulate_branch.o \
              arm64_emulate_immediate.o \
              arm64_emulate_ldst.o \
              arm64_emulate_register.o \
              arm64_emulate_simd.o

# 主入口：lsdriver.c 编译为 lsdriver_main.o
$(obj)/lsdriver_main.o: $(src)/lsdriver.c FORCE
	$(call if_changed_dep,cc_o_c)

# O3 优化
ccflags-y += -O3

# Android 12 / 5.10 上先不启用模块级 full LTO，避免与内核 LTO=thin 冲突
# ccflags-y += -flto

# Debug 开关
# ccflags-y += -DDEBUG

# Android 12 GKI 5.10 的 Clang(12/14) 对 +bf16/+i8mm 支持不完整，降到 armv8.5-a
ccflags-y += -march=armv8.5-a+fp16+fp16fml+lse+rcpc+crc

# 禁止优化器主动向量化
ccflags-y += -fno-vectorize -fno-slp-vectorize

# 仅从显式使用 FP/SIMD 汇编的目标中移除 -mgeneral-regs-only
CFLAGS_REMOVE_lsdriver_main.o += -mgeneral-regs-only
CFLAGS_REMOVE_emulate_inst.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate_ldst.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate_immediate.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate_register.o += -mgeneral-regs-only
CFLAGS_REMOVE_arm64_emulate_simd.o += -mgeneral-regs-only

# ---- 针对 arm64_emulate_simd.o 的特殊处理 ----
# 1) 该文件使用 usdot 内联汇编（i8mm 扩展），单独为它开启 i8mm。
#    只影响这一个文件，不会让整个模块都要求 CPU 支持 i8mm。
CFLAGS_arm64_emulate_simd.o += -march=armv8.5-a+i8mm

# 2) 该文件若参与 ThinLTO，LLVM 后端处理 usdot 时会崩溃，
#    因此对它单独禁用 LTO；同时必须一并禁用 CFI，否则会出现
#    "invalid argument '-fsanitize=cfi' only allowed with '-flto'"。
CFLAGS_REMOVE_arm64_emulate_simd.o += -flto=thin
CFLAGS_REMOVE_arm64_emulate_simd.o += -fsanitize=cfi
CFLAGS_arm64_emulate_simd.o       += -fno-lto -fno-sanitize=cfi
# -----------------------------------------------------

CFLAGS_lsdriver_main.o += -mno-implicit-float
CFLAGS_emulate_inst.o += -mno-implicit-float
CFLAGS_arm64_emulate_ldst.o += -mno-implicit-float
CFLAGS_arm64_emulate_simd.o += -mno-implicit-float
CFLAGS_arm64_emulate_immediate.o += -mno-implicit-float
CFLAGS_arm64_emulate_register.o += -mno-implicit-float

# 警告不视为错误
ccflags-y += -Wno-error

# 现代 GNU C 方言
ccflags-y += -std=gnu11

# 关闭栈保护
ccflags-y += -fno-stack-protector

# 省略帧指针
ccflags-y += -fomit-frame-pointer

# 尾调用优化
ccflags-y += -foptimize-sibling-calls

# 注意：Android 12 GKI 5.10 启用了 BTI/PAC，绝不能加 -mbranch-protection=none
# （原注释里描述的 CFI jump table / BTI panic 是 6.1+ 的坑，5.10 无 KCFI）
#ccflags-y += -mbranch-protection=none

# 关闭自动局部变量清零
ccflags-y += -ftrivial-auto-var-init=uninitialized

# 循环展开
ccflags-y += -funroll-loops

# 严格别名优化
ccflags-y += -fstrict-aliasing

# 放宽本模块栈帧上限
ccflags-y += -Wframe-larger-than=4096

# 关闭各类 sanitizer 插桩
KASAN_SANITIZE := n
UBSAN_SANITIZE := n
KCSAN_SANITIZE := n
KMSAN_SANITIZE := n
GCOV_PROFILE := n
KCOV_INSTRUMENT := n
