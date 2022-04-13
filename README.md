# chengqigang.github.io
doc

@startuml 入库

header **<size:15>维融科技股份有限公司</size>**
footer **<size:15>author 程启罡 </size>**

title  入库

autonumber

actor 用户 as USER
collections 管理系统 as BROWSER
collections 消息通知 as MQ
collections 业务系统 as SERVICE
collections 控制中心 as CONTROL
collections 设备 as DEVICE

USER -> BROWSER : 入库
activate BROWSER

USER -> BROWSER : 选择 产品类型、\n产品规格、数量 
USER -> BROWSER : 确认入库

BROWSER -> SERVICE : 入库
activate SERVICE

SERVICE -> SERVICE : 校验能否入库

alt 不能入库
    BROWSER <- SERVICE : 不能入库
    deactivate SERVICE
    USER <- BROWSER : 不能入库
    deactivate BROWSER
else 可以入库
    activate SERVICE
    SERVICE -> CONTROL : 入库(产品类型、\n产品规格、数量)
    activate CONTROL
    activate BROWSER

    BROWSER <- SERVICE : 正在入库，请稍后
    deactivate SERVICE

    loop 托盘入库批次循环
        CONTROL -> CONTROL : 弹出托盘

        SERVICE <- CONTROL : 托盘已弹出
        activate MQ
        BROWSER <- SERVICE : 托盘已弹出，请放入物品
        deactivate MQ

        USER -> DEVICE : 放入物品
        USER -> BROWSER : 确认入库
        BROWSER -> SERVICE : 确认入库
        activate SERVICE
        SERVICE -> CONTROL : 入库
        BROWSER <- SERVICE : 正在识别，请稍后
        deactivate SERVICE

        loop 托盘格识别循环
            CONTROL -> CONTROL : 格子拍照、识别
            alt 格子识别异常
                SERVICE <- CONTROL : XXX位置产品异常
                
                activate MQ
                BROWSER <- SERVICE : XXX位置产品有异常，请处理
                deactivate MQ

                alt 图片与识别编号不一致(用户手动修改编号)
                    USER -> BROWSER : 用户手动修改编号,图片与识别编号不一致
                    BROWSER -> SERVICE : 修改识别编号
                    activate SERVICE
                    SERVICE -> CONTROL : 修改识别编号
                    BROWSER <- SERVICE : 正在识别，请稍后
                    deactivate SERVICE

                else 图片与识别编号一致(放入物品异常)
                    USER -> BROWSER : 图片与识别编号一致
                    BROWSER -> SERVICE : 暂不处理
                    activate SERVICE
                    SERVICE -> CONTROL : 暂不处理
                    BROWSER <- SERVICE : 正在识别，请稍后
                    deactivate SERVICE
                end
            end
            SERVICE <- CONTROL : 识别结果上送
        end

        CONTROL -> CONTROL : 托盘识别结束，是否\n有暂不处理的异常

        alt 有暂不处理的异常
            CONTROL -> CONTROL : 弹出托盘
            SERVICE <- CONTROL : 托盘已弹出
            activate MQ
            BROWSER <- SERVICE : 请核对XXX位置的物品
            deactivate MQ
            USER -> BROWSER : 确认
            BROWSER -> SERVICE : 处理XXX位置物品
            activate SERVICE
            SERVICE -> CONTROL : 处理XXX位置物品
            BROWSER <- SERVICE : 正在识别，请稍后
            deactivate SERVICE
            CONTROL -> CONTROL : 重新识别XXX位置
            SERVICE <- CONTROL : 识别结果上送
        end
        
        CONTROL -> CONTROL : 是否还有剩余批次
        alt 无剩余批次
            SERVICE <- CONTROL : 入库成功
            deactivate CONTROL
            
            activate MQ
            BROWSER <- SERVICE : 入库成功
            deactivate MQ
            USER <- BROWSER : 入库成功
            deactivate BROWSER
        end
    end
end


deactivate BROWSER


@enduml
