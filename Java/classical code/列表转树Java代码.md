```
    private List<ITOrgVO> list2tree(List<ITOrgVO> list){
        Map<String,ITOrgVO> map = list.stream().collect(
                Collectors.toMap(ITOrgVO::getId,ITOrgVO-> ITOrgVO,(oldValue,newValue)-> newValue )
        );
        List<ITOrgVO> resultList = new ArrayList<>();
        for(ITOrgVO itOrgVO : list){
            String pid = itOrgVO.getParentId();
            if(pid != null){
                ITOrgVO parentVO = map.get(pid);
                if(parentVO != null){
                    List<ITOrgVO> children = parentVO.getChildren();
                    if(children != null){
                        children.add(itOrgVO);
                    }else{
                        children = new ArrayList<>();
                        parentVO.setChildren(children);
                        children.add(itOrgVO);
                    }
                }else{
                    resultList.add(itOrgVO);
                }
            }else{
                resultList.add(itOrgVO);
            }
        }
        return resultList;
    }
```

