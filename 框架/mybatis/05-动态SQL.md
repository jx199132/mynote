# if条件
## 测试代码

```
@Test
	public void testSearchStudents(){
		logger.info("根据条件查询学生列表");
		Map<String, Object> map = new HashMap<String, Object>();
		map.put("name", "李%");
		map.put("age", 11);
		List<Student> studentList = studentMapper.searchStudents(map);
		for(Student s : studentList){
			System.out.println(s);
		}
	}
```
## java Mapper
```
List<Student> searchStudents(Map<String, Object> map);
```
## 配置文件 Mapper

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.java1234.mappers.StudentMapper">

	 <resultMap type="Student" id="StudentResult">
		<id property="id" column="id"/>
		<result property="name" column="name"/>
		<result property="age" column="age"/>
	</resultMap>
	
	<select id="searchStudents" parameterType="map" resultMap="StudentResult">
		select * from t_student
		where 1=1
		<if test="name != null">
			and name like #{name}
		</if>
		<if test="age != 0">
			and age = #{age}
		</if>
	</select>
</mapper>
```
# choose，when和otherwise条件

```
<select id="searchStudents2" parameterType="Map" resultMap="StudentResult">
		select * from t_student 
		 <choose>
		 	<when test="searchBy=='gradeId'">
		 		where gradeId=#{gradeId}
		 	</when>
		 	<when test="searchBy=='name'">
		 		where name like #{name}
		 	</when>
		 	<otherwise>
		 		where age=#{age}
		 	</otherwise>
		 </choose>
		 
	</select>
```
# where条件

```
<select id="searchStudents3" parameterType="Map" resultMap="StudentResult">
		select * from t_student 
		 <where>
			 <if test="gradeId!=null">
			 	gradeId=#{gradeId}
			 </if>
			 <if test="name!=null">
			 	and name like #{name}
			 </if>
			 <if test="age!=nulll">
			 	and age=#{age}
			 </if>
		 </where>
	</select>
```
# trim条件
```
<select id="searchStudents4" parameterType="Map" resultMap="StudentResult">
		select * from t_student 
		 <trim prefix="where" prefixOverrides="and|or">
			 <if test="gradeId!=null">
			 	gradeId=#{gradeId}
			 </if>
			 <if test="name!=null">
			 	and name like #{name}
			 </if>
			 <if test="age!=nulll">
			 	and age=#{age}
			 </if>
		 </trim>
	</select>
```
# foreach循环
会自动剔除最后一个逗号
```
<select id="searchStudents5" parameterType="Map" resultMap="StudentResult">
		select * from t_student 
		 <if test="gradeIds!=null">
		 	<where>
		 		gradeId in 
		 		<foreach item="gradeId" collection="gradeIds" open="(" separator="," close=")">
		 		 #{gradeId}
		 		</foreach>
		 	</where>
		 </if>
	</select>
```
# set条件
会自动剔除最后一个逗号
```
<update id="updateStudent" parameterType="Student">
		update t_student
		<set>
		 <if test="name!=null">
		 	name=#{name},
		 </if>
		 <if test="age!=null">
		 	age=#{age},
		 </if>
		</set>
		where id=#{id}
	</update>
```
