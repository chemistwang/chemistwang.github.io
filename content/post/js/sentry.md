









// 二进制流导出版本2
export const StreamDownloadV2 = (res: any) => {
    const { headers } = res;
    const content = headers['content-disposition'];
    //eg: 'attachment;filename=%E5%BE%85%E5%AE%A1%E6%A0%B8%E5%B7%A5%E4%BD%9C%E4%BA%BA%E5%91%98%E5%AF%BC%E5%87%BA.xlsx
    const filename = decodeURIComponent(content.split(';')[1].split('=')[1]);
    const url = window.URL.createObjectURL(new Blob([res.data]));
    var link = document.createElement('a');
    link.style.display = 'none';
    link.href = url;
    link.setAttribute('download', filename)
    // window.URL.revokeObjectURL(url);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}

二进制流下载 444..png 会有下划线





  Line 9:9:  'React' must be in scope when using JSX  react/react-in-jsx-scope

  16 版本的React写函数组件时需要引入 // import React from 'react'



  