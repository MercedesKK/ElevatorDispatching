<template>
    <div class="myFramework" aria-label="A complete example of page header">
        <el-page-header>
            <template #breadcrumb>
                <el-breadcrumb separator="/">
                    <el-breadcrumb-item>
                        电梯调度作业
                    </el-breadcrumb-item>
                </el-breadcrumb>
            </template>
            <template #content>
                <div class="flex items-center">
                    <img class="my_photo" src="../assets/myPhoto.png" />
                    <span class="text-large font-600 mr-3"> MercedesKK </span>
                </div>
            </template>


            <el-descriptions :column="3" size="small" class="mt-4 padding_left">
                <!-- <div class="row">
                    <div class="col-4"></div>
                </div> -->


                <el-descriptions-item label="Username">张靖凯</el-descriptions-item>
                <el-descriptions-item label="School">同济大学软件学院</el-descriptions-item>
                <el-descriptions-item label="Email">2455650395@qq.com</el-descriptions-item>

            </el-descriptions>
            <p class="mt-4 text-sm">
            </p>
        </el-page-header>
    </div>
</template>

<script>

</script>

<style scoped>
.myFramework {
    padding-top: 2vh;
}

.padding_left {
    padding-left: 10vw;
}

.my_photo {
    border-radius: 30px;
    width: 40px;
    height: 40px;
}
</style>