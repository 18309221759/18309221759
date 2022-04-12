### Hi there 👋

<!--
**18309221759/18309221759** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
```
# ---------------------------------------------------------------
# Include the product definitions.  包含了产品定义
# We need to do this to translate TARGET_PRODUCT into its
# underlying TARGET_DEVICE before we start defining any rules. # 在我们开始定义任何规则之前，我们需要这样做以将 TARGET_PRODUCT 转换为其底层 TARGET_DEVICE。 
#
include $(BUILD_SYSTEM)/node_fns.mk
include $(BUILD_SYSTEM)/product.mk
include $(BUILD_SYSTEM)/device.mk

ifneq ($(strip $(TARGET_BUILD_APPS)),)  #如果TARGET_BUILD_APPS不等于空
# An unbundled app build needs only the core product makefiles.  #未捆绑的应用程序构建只需要 核心产品 makefile
all_product_configs := $(call get-product-makefiles,\   # get-product-makefiles 在 build/core/product.mk 里面定义
    $(SRC_TARGET_DIR)/product/AndroidProducts.mk)       # 查找的文件是 ./build/core/config.mk:54:SRC_TARGET_DIR := $(TOPDIR)build/target
else     #如果TARGET_BUILD_APPS 等于 空
# Read in all of the product definitions specified by the AndroidProducts.mk
# files in the tree.       # 读入树中 AndroidProducts.mk 文件指定的所有产品定义。 
all_product_configs := $(get-all-product-makefiles)    # get-all-product-makefiles 在 build/core/product.mk 里面定义
endif
=============================================================
>>>product.mk文件<<<<
# Functions for including AndroidProducts.mk files  #包含AndroidProduct.mk文件的函数
# PRODUCT_MAKEFILES is set up in AndroidProducts.mks. #PRODUCT_MAKEFILES在AndroidProducts.mk文件里面安装
# Format of PRODUCT_MAKEFILES:  # PRODUCT_MAKEFILES的格式
# <product_name>:<path_to_the_product_makefile>   #a:b型
# If the <product_name> is the same as the base file name (without dir  # 如果a与product makefile的基本文件名（没有目录和.mk后缀）相同，
# and the .mk suffix) of the product makefile, "<product_name>:" can be  # "a:"可以忽略
# omitted.

#
# Returns the list of all AndroidProducts.mk files.  #返回所有AndroidProducts.mk文件的列表
# $(call ) isn't necessary.
#
define _find-android-products-files
$(shell test -d device && find device -maxdepth 6 -name AndroidProducts.mk) \
  $(shell test -d vendor && find vendor -maxdepth 6 -name AndroidProducts.mk) \
  $(SRC_TARGET_DIR)/product/AndroidProducts.mk
endef
#
# Returns the sorted concatenation of PRODUCT_MAKEFILES
# variables set in the given AndroidProducts.mk files.  #返回给定 AndroidProducts.mk 文件中设置的 PRODUCT_MAKEFILES 变量的排序连接。
# $(1): the list of AndroidProducts.mk files.
#
define get-product-makefiles
$(sort \
  $(foreach f,$(1), \
    $(eval PRODUCT_MAKEFILES :=) \
    $(eval LOCAL_DIR := $(patsubst %/,%,$(dir $(f)))) \
    $(eval include $(f)) \
    $(PRODUCT_MAKEFILES) \
   ) \
  $(eval PRODUCT_MAKEFILES :=) \
  $(eval LOCAL_DIR :=) \
 )
endef
#
# Returns the sorted concatenation of all PRODUCT_MAKEFILES
# variables set in all AndroidProducts.mk files.  #返回所有 AndroidProducts.mk 文件中设置的所有 PRODUCT_MAKEFILES 变量的排序连接。
# $(call ) isn't necessary.
#
define get-all-product-makefiles
$(call get-product-makefiles,$(_find-android-products-files))
endef

=============================================================

# Find the product config makefile for the current product.  #查找当前产品的产品配置makefile
# all_product_configs consists items like:  # all_product_configs 包含以下项目：
# <product_name>:<path_to_the_product_makefile>   #形如a:b形式,eg: aosp_manta:device/samsung/manta/aosp_manta.mk
# or just <path_to_the_product_makefile> in case the product name is the  
# same as the base filename of the product config makefile.    # 或只是 <path_to_the_product_makefile> 以防产品名称与产品配置 makefile 的基本文件名相同。 
current_product_makefile :=  #先给这两个变量清空，下面要用到这两个变量存储内容
all_product_makefiles :=
$(foreach f, $(all_product_configs),\
    $(eval _cpm_words := $(subst :,$(space),$(f)))\ #如果f=a:b,将f内容改变为a b     #  如果f=a, 那么没有替换，f的内容不变，为a
    $(eval _cpm_word1 := $(word 1,$(_cpm_words)))\  # 将改变后f的a赋值给_cpm_word1  #  将f的内容a赋值给_cpm_word1
    $(eval _cpm_word2 := $(word 2,$(_cpm_words)))\  # 将改变后f的b赋值给_cpm_word2  #  因为f只有一个单词，所以_cpm_word2为空
    $(if $(_cpm_word2),\  
        $(eval all_product_makefiles += $(_cpm_word2))\ #如果_cpm_word2有内容，a:b型
        $(if $(filter $(TARGET_PRODUCT),$(_cpm_word1)),\
            $(eval current_product_makefile += $(_cpm_word2)),),\
        $(eval all_product_makefiles += $(f))\           #如果_cpm_word2无内容， a型
        $(if $(filter $(TARGET_PRODUCT),$(basename $(notdir $(f)))),\   #$(basename $(nodir $(f)))作用：如果f=device/samsung/manta/aosp_manta.mk，则输出aosp_manta
            $(eval current_product_makefile += $(f)),)))
_cpm_words :=  #使用完清空这三个变量
_cpm_word1 :=
_cpm_word2 :=
current_product_makefile := $(strip $(current_product_makefile))
all_product_makefiles := $(strip $(all_product_makefiles))

```
