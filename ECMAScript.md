
## [Bootstrap] 
### Alert 
使用ajax 显示 Alert
   $.ajax({
            url: 'ajax-route-save',
            type: "POST",
            data: {'uuid':uuid,'start':rsStart,'end':rsEnd},
         //   dataType: "json",
            async: false,
            success: function (json) {
                //datas = json.data;
                $(".tip").html('<div class="alert alert-success"> <button type="button" class="close"      data-dismiss="alert">×</button> <strong> <i class="fa fa-check-circle fa-lg fa-fw"></i> 成功. </strong> </div>');
            }
        });


## [jQuery]
