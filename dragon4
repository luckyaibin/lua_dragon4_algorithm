local BigNumber = {
len=1;
int32s = {0};
radix = 10000000;
max_safe_int = 9007199254740991;
};
BigNumber.__index = BigNumber;

function BigNumber:new(int32s)
	local self = {};
	setmetatable(self,BigNumber);
	self.len=1;
	self.int32s={};
	self.radix = 10000000;
	--self.max_safe_int = 9007199254740991;
	if int32s then
		if type(int32s) == "string" then
			assert(nil,'无效的输入，暂不支持');
		elseif type(int32s) == "table" then
			local len = #int32s;
			for i=len,1,-1 do
				self.int32s[i] = int32s[len-i+1];
			end
			self.len = #(int32s);
		end
	else
		self.int32s={0};
	end
	return self;
end


function BigNumber:copy(o)
	self.int32s = {};
	for k,v in pairs(o.int32s) do
		table.insert(self.int32s,v);
	end
	self.len = o.len;
end
--移位操作：基数是24bit，
--最大可表示的整数是53bit
-- 11111 11111111 11111111 11111111 11111111 11111111 11111111 BigNumber.max_safe_int,	53bits
--                                  10011000 10010110 10000000 BigNumber.radix			24bit
-- 11111 11111111 11111111 11111111 (536870911),29bit
function shift_left(lhs,shift)
	if shift == 0 then
		return;
	elseif shift >= 24 then--先整体移动24bit，在移动剩下的不足32bit的
		local skip = math.floor(shift / 24);
		for i=lhs.len,1 do
			lhs.int32s[i+skip] = lhs.int32s[i];
		end
		for i=1,skip do
			lhs.int32s[i] = 0;
		end
		lhs.len = #lhs.int32s;
		shift_left_less24(lhs,shift % 24);
	else
		shift_left_less24(lhs,shift);
	end
end

function shift_left_less24(lhs,shift)
	if shift == 0 then
		return;
	else
		local mod = 2^shift;
		local int32OverFlow = BigNumber.radix;
		local int32 = 0;
		local carry = 0;
		for i=1,lhs.len do
			int32 = lhs.int32s[i];
			int32 = int32*mod + carry;--移位
			if int32 >= int32OverFlow then
				carry = math.floor(int32 / int32OverFlow);
				int32 = int32 % int32OverFlow ;
			else
				carry = 0;
			end
			lhs.int32s[i] = int32;
		end
		if carry > 0 then
			lhs.int32s[lhs.len+1] = carry;
		end
		lhs.len = #lhs.int32s;
	end
end

-- A B C D ...
function shift_right(lhs,shift)
	if shift == 0 then
		return;
	elseif shift >= 24 then--先整体移动24bit，在移动剩下的不足32bit的
		local skip = math.floor(shift / 24);
		for i=1 ,lhs.len - skip do --缩短了
			lhs.int32s[i]  = lhs.int32s[i+skip];
		end
		--去掉尾巴
		for i=lhs.len,lhs.len-skip+1,-1 do
			lhs.int32s[i] = nil;
		end
		lhs.len = #lhs.int32s;
		shift_right_less24(lhs,shift % 24);
	else
		shift_right_less24(lhs,shift);
	end
end

function shift_right_less24(lhs,shift)
	local remainder = 0;
	local simplenum = 2^shift;
	local radix = BigNumber.radix;
	local digits = {};
	for i=lhs.len,1,-1 do
		local int32 = lhs.int32s[i] + remainder*radix;
		local digit = math.floor(int32/simplenum);
		remainder = int32 % simplenum;
		if #digits == 0 and digit == 0 then
		else
			table.insert(digits,digit);
		end
	end
	if #digits == 0 then
		digits = {0};
	end
	--
	lhs.int32s = {};
	for k,v in pairs(digits) do
		lhs.int32s[k] = v;
	end
	lhs.len = #lhs.int32s;
end

function shift_right_less24_(lhs,shift)
	if shift == 0 then
		return;
	else
		local mod = 2^shift;
		local shift_exp = 2^(24-shift);
		local int32_lo = 0;
		local int32_hi = 0;
		local carry = 0;
		for i=1,lhs.len - 1 do
			int32_lo = math.floor(lhs.int32s[i] / mod);
			int32_hi = math.floor((lhs.int32s[i+1] % mod) * BigNumber.radix / shift_exp);
			local int32 = int32_lo + int32_hi;
			lhs.int32s[i] = int32;
		end
		local highest = math.floor(lhs.int32s[lhs.len] / mod);
		if highest ~= 0 then
			lhs.int32s[lhs.len] = highest
		else
			lhs.int32s[lhs.len] = nil
		end
		
		lhs.len = #lhs.int32s;
	end
end

function add(lhs,rhs)
	local result = BigNumber:new();
	local int32OverFlow = BigNumber.radix;
	local int32s = {};
	local len = lhs.len > rhs.len and lhs.len or rhs.len;
	local carry = 0;
	for i=1,len do
		local int32_lhs = lhs.int32s[i] or 0;
		local int32_rhs = rhs.int32s[i] or 0;
		local r = int32_lhs + int32_rhs + carry;
		if r >= int32OverFlow then
			r = r % int32OverFlow;
			carry = 1;
		else
			carry = 0;
		end
		table.insert(int32s,r);
	end
	
	result.int32s = int32s;
	result.len = #int32s;
	return result;
end

function __addin32_at_index(lhs,index,int32)
	local int32OverFlow = BigNumber.radix;
	if index > lhs.len then--先扩充
		for i=lhs.len+1,index do
			lhs.int32s[i] = 0;
		end
		lhs.len = index;
	end
	local carry = int32;
	for i=index,lhs.len do
		local int32_lhs = lhs.int32s[i]
		local r = int32_lhs + carry;
		if r >= int32OverFlow then
			r = r % int32OverFlow;
			carry = 1;
		else
			carry = 0;
		end
		lhs.int32s[i] = r;
		if(carry==0)then--没有进位，直接结束
			break;
		end
	end
	if carry > 0 then--最高位进位了
		--print('最高位进位了！')
		lhs.int32s[lhs.len+1] = carry;
	end
	lhs.len = #lhs.int32s;
	return lhs;
end



--[[
a	b	c
x	   d	e
-------------
ea  eb  ec
da db  dc


]]

function mult(lhs,rhs)
	local small,big;
	if lhs.len < rhs.len then
		small = lhs;
		big = rhs;
	else
		small = rhs;
		big = lhs;
	end
	local int32OverFlow = BigNumber.radix;
	local result = BigNumber:new();
	
	local multi_carry = 0;
	for i=1,small.len do
		local s = small.int32s[i];
		if s ~= 0 then
			for j=1,big.len do
				local b = big.int32s[j];
				local r = s * b;
				assert(r <= BigNumber.max_safe_int,'太大的结果导致不准确了！')
				if r >= int32OverFlow then
					multi_carry = math.floor( r / int32OverFlow);
					r = r - multi_carry * int32OverFlow;
				else
					multi_carry = 0;
				end
				local index = i + j - 1;
				__addin32_at_index(result,index,r)
				if multi_carry > 0 then
					__addin32_at_index(result,index+1,multi_carry)
				end
			end
		end
	end
	for k,v in pairs(result.int32s) do
		assert( v < int32OverFlow,'over flow')
	end
	result.len = #result.int32s;
	return result;
end


function getBigNumberDecimal(o)
	local str = '';
	if type(o) == 'number' then
		return tostring(o);
	else
		for i=1,o.len do
			--str = int32ToBits(o.int32s[i]) .. ' ' .. str;
			local bits = tostring(o.int32s[i]);
			if string.len(bits) < 7 and i < o.len then
				bits = string.rep('0',7-string.len(bits)) .. bits;
			end
			str = bits .. '' .. str;
		end
		return str;
	end
	
end

function compare(lhs,rhs)
	if lhs.len == rhs.len then
		for i=lhs.len,1,-1 do
			local l = lhs.int32s[i];
			local r = rhs.int32s[i];
			if l ~= r then
				if l < r then
					return -1;
				else
					return 1;
				end
			end
		end
		return 0;
	else
		if lhs.len < rhs.len then
			return -1;
		else
			return 1;
		end
	end
end


--两个数字低位对齐相减
-- lhs : a b c d e f g
-- rhs :       h  i  j k
function __div_minus_at(u,v,index)
	local borrow = 0;
	local radix = BigNumber.radix;
	for  i=1,v.len do
		local uu = u.int32s[index + i - 1] + borrow;
		local vv = v.int32s[i];
		local digit=0
		if uu-vv < 0 then
			borrow = -1;
			digit=radix + uu-vv
		else
			borrow = 0;
			digit=uu-vv
		end
		u.int32s[index + i - 1] = digit
	end
	if borrow==-1 then--最高位出现了借位
		return true
	else
		return false
	end
end

function __div_plus_at(lhs,rhs,index)
	local carry = 0;
	local radix = BigNumber.radix;
	for  i=1,rhs.len do
		local u = lhs.int32s[index + i - 1];
		local v = rhs.int32s[i] + carry;
		local digit= u+v+carry;
		if digit >= radix then
			carry = 1;
			digit=digit-radix
		else
			carry = 0;
		end
		lhs.int32s[index + i - 1] = digit
	end
	if carry== 1 then--最高位出现了进位
		return true
	else
		return false
	end
end

function div_simple(bignum,simplenum)
	local remainder = 0;
	local radix = BigNumber.radix;
	local digits = {};
	for i=bignum.len,1,-1 do
		local int32 = bignum.int32s[i] + remainder*radix;
		local digit = math.floor(int32/simplenum);
		remainder = int32 % simplenum;
		if #digits == 0 and digit == 0 then
		else
			table.insert(digits,digit);
		end
	end
	if #digits == 0 then
		digits = {0};
	end
	return BigNumber:new(digits),BigNumber:new({remainder});
end
 
-- 计算 lhs / rhs
function div(u,v)
	local radix = BigNumber.radix;
	local half_radix = 5000000;
	local r = compare(u,v);
	--print("比较结果:",r);
	if v.len == 1 then
		return div_simple(u,v.int32s[1]);
	else
		
		local u_old_len = u.len;
		local vn = v.int32s[v.len];
		--保证v最高位是大于等于math.floor(BigNumb.radix/2)
		local d = 1;
		if vn < half_radix then
			d = math.floor(radix / (vn + 1));
			--print('d is ',d)
			local fac = BigNumber:new({d})
			u = mult(u,fac);
			v = mult(v,fac);
		end
		if u_old_len == u.len then --没增长,最前面增加0
			u.int32s[u.len+1] = 0;
			u.len = u.len + 1;
		end
		local digits = {};
		local q = 0;
		for m=u.len,v.len+1,-1 do
			local u_high12 = u.int32s[m] * radix + u.int32s[m-1];
			local u_high3 = 0;
			if m-2 >=1 then
				u_high3 = u.int32s[m-2];
			end
			local v_high1 = v.int32s[v.len];
			local v_high2 = v.int32s[v.len-1];
			local q_guess = math.floor(u_high12 / v_high1);
			local r_guess = u_high12 % v_high1;
			while q_guess == radix or q_guess*v_high2 > radix*r_guess + u_high3 do
				q_guess = q_guess - 1;
				r_guess = r_guess + v_high1;
			end
			q = q_guess;
			--D4
			local Q = BigNumber:new({q});
			--底部对齐来相减（因为可能q*v之后的Q比q长度+1)
			local highest_is_negative =__div_minus_at(u,mult(Q,v),m - v.len)
			if highest_is_negative then --小概率事件
				q = q - 1;
				__div_plus_at(u,v,m - v.len)
			else
				
			end
			--不插入高位的0
			if #digits == 0 and q == 0 then
			else
				table.insert(digits,q);
			end
		end
		
		local R,R_of_R = div_simple(u,d)
		assert(R_of_R.int32s[1] == 0,'错误');
		
		if #digits == 0 then
			digits = {0};
		end
		local Q = BigNumber:new(digits);
		return Q,R
	end
	
end
 


function test_div()
	for test_num = 0,100 do
		local u_len = math.random(3,4);
		local v_len = math.random(3,4);
		local r_len = math.random(0,2);
		local u_int32s = {};
		local v_int32s = {};
		local r_int32s = {};
		for i=1,u_len do
			local U = math.random(0,5000000*2-1);
			table.insert(u_int32s,U);
		end
		for i=1,v_len do
			local V = math.random(0,5000000*2-1);
			table.insert(v_int32s,V);
		end
		
		local u = BigNumber:new(u_int32s);
		local v = BigNumber:new(v_int32s);
		
		local UXV= mult(u,v);
		
		for i=0,r_len do
			local R = math.random(0,2^8-1);
			table.insert(r_int32s,R);
		end
		
		local r = BigNumber:new(r_int32s);
		UXV = add(UXV,r);
		
		local UXV_str = getBigNumberDecimal(UXV)
		--print('UXV-------:',UXV_str)
		--print('u---------:',getBigNumberDecimal(u))
		--print('v---------:',getBigNumberDecimal(v))
		--print('r:',getBigNumberDecimal(r))
		
		
		local Q,R = div(UXV,u);
		print('Q:',getBigNumberDecimal(Q))
		print('R:',getBigNumberDecimal(R))
		local tt = mult(Q,u);
		local ss = add(tt,R);
		local ss_str = getBigNumberDecimal(ss)
		--print('ss-------:',ss_str)
		--print('=============')
		
		assert(UXV_str == ss_str,'結果不一致')
		--print("\n\n")
	end
end

function test_div2()
	local u = BigNumber:new({7239458,1234124,412934});
	local v = BigNumber:new({8039458,1234124,412934});
	
	local Q,R = div(u,v);
	print('Q:',getBigNumberDecimal(Q))
	print('R:',getBigNumberDecimal(R))
	
	local uu = add(mult(v,Q),R);
	print('uu:',getBigNumberDecimal(uu))
end
 




function test_shift()
	for i=1,100 do
		local bignum_len = math.random(1,100);
		local bignum = nil;
		local bignumcopied = nil;
		for j=1,bignum_len do
			local nums = {};
			local num = math.random(0,BigNumber.radix-1);
			table.insert(nums,num);
			bignum =  BigNumber:new(nums);
			bignumcopied = BigNumber:new(nums);
		end
		local shift_bits = math.random(0,250);
		
		local bits = getBigNumberDecimal(bignum);
		shift_left(bignumcopied,shift_bits)
		--print("after shiftleft:",getBigNumberDecimal(bignumcopied));
		shift_right(bignumcopied,shift_bits);
		--print("after shiftright:",getBigNumberDecimal(bignumcopied));
		local bits2 = getBigNumberDecimal(bignumcopied);
		
		if bits ~= bits2 then
			print('original value', getBigNumberDecimal(bignum))
			print('shift_bits',shift_bits);
			print(bits);
			print(bits2);
		end
		
	end
end


function int8ToBits(num)
	num=math.floor(num);
	local bits = '';
	for i=8,1,-1 do
		local bit = 0;
		if num % 2 > 0 then
			bit = 1;
		end
		bits = bit .. bits;
		num = math.floor(num / 2);
	end
	return bits;
end

function int32ToBits(num)
	num=math.floor(num);
	local bits = '';
	for i=32,1,-1 do
		local bit = 0;
		if num % 2 > 0 then
			bit = 1;
		end
		bits = bit .. bits;
		num = math.floor(num / 2);
	end
	return bits;
end

function bitsToInt32(bits)
	local i=1;
	bits = string.gsub(bits,' ','');
	local j=#bits;
	local intValue = 0;
	for idx = 1,j do
		local bit = string.sub(bits,idx,idx);
		bit = tonumber(bit);
		intValue = intValue * 2;
		intValue = intValue + bit;
	end
	return intValue;
end

function int64ToBits(num)
	local i=1;
	bits = string.gsub(bits,' ','');
	local j=#bits;
	local intValue = 0;
	for idx = 1,j do
		local bit = string.sub(bits,idx,idx);
		bit = tonumber(bit);
		intValue = intValue * 2;
		intValue = intValue + bit;
	end
	return intValue;
end

function bitsToInt64(bits)
	assert(nil,'不能这样用');
end

--bit字符串对应的int,longlong值转成string里去
function bitsToString(bits)
	if type(bits) == 'table' then
		assert(nil,'还没实现')
	elseif type(bits) == 'string' then
		bits = string.gsub(bits,' ','');--去掉空格
		--十六进制的字符串直接转成二进制字符串算了。。难得麻烦
		if string.sub(bits,1,1) == '0' and (string.sub(bits,2,2) == 'x' or string.sub(bits,2,2) == 'X') then
			local bitOfHex = '';
			for i=3,#bits do
				local h = string.sub(bits,i,i);
				local HexBitMap = {
				['0'] = '0000';['1'] = '0001';['2'] = '0010';['3'] = '0011';['4'] = '0100';['5'] = '0101';['6'] = '0110';['7'] = '0111';['8'] = '1000';	['9'] = '1001';
				['a'] = '1010';['b'] = '1011';['c'] = '1100';['d'] = '1101';['e'] = '1110';['f'] = '1111';
				['A'] = '1010';['B'] = '1011';['C'] = '1100';['D'] = '1101';['E'] = '1110';['F'] = '1111';}
				local bit_str = HexBitMap[h];
				assert(bit_str,'invalid char in hex string。');
				bitOfHex = bitOfHex .. bit_str;
			end
			--已转成了0/1字符串
			bits = bitOfHex;
		end
		
		local len = #bits;
		
		local left = len % 8;
		if left ~= 0 then--凑足8的倍数
			local pre_zeros = string.rep('0',8-left);
			bits = pre_zeros .. bits;
		end
		
		local len = #bits;
		local bytenum = len / 8;
		local str = '';
		for i=1,bytenum do
			local intValue = 0;
			for j = (i-1) * 8 + 1, i*8 do --取出8个0/1字符串的每一个
				local bit = string.sub(bits,j,j);
				if bit == '0' then
					intValue = intValue + 0;
				elseif bit == '1' then
					intValue = intValue + 1;
				else
					assert(ni,'无效的字符'..bit)
				end
				if j < i*8 then
					intValue = intValue * 2;
				end
			end
			local c = string.char(intValue);
			str = str .. c;
		end
		return str;
	end
end

--序列化在字符串里的二进制值，转成二进制字符串
function stringToBits(bitsInString)
	if type(bitsInString) == 'string' then
		local concat = function (lst)
			local str='';
			for i=0,#lst do
				str = str .. lst[i];
			end
			return str;
		end
		local bits = '';
		for i=1,#bitsInString do
			local c = string.sub(bitsInString,i,i);--每次取出一个char来处理
			local b = string.byte(c);
			local bits_arr = int8ToBits(b);
			if bits ~= '' then
				bits = bits .. ' ';
			end
			bits_arr = concat(bits_arr)
			if #bits_arr < 8 then
				bits_arr = string.rep('0', 8 - #bits_arr) .. bits_arr;
			end
			bits = bits .. bits_arr;
		end
		return bits;
	end
end
function ten_exp(e)
	assert(e < 1024,'too big exponent of ten')
	local result = BigNumber:new({1});
	if e == 0 then
		return result
	end
	local index = 0
	while(e > 0) do
		if e % 2 == 1 then
			result = mult(result,ten_2exp_lookup[index]);
		end
		index = index + 1;
		e = math.floor(e/2);
	end
	return result;
end

--将输出表示为最接近的10进制小数
function dragon4(exponent,mantissa)
	local valueMantissa = 0;
	local valueExponent = 0;
	local value = 0;	
	local valueMarginLow;
	local valueMarginHigh;
	--Denormalized float
	if exponent == 0 then
		--value = (mantissa / 2^23) * (2 ^ (1-127));
		valueMantissa = mantissa;
		valueExponent = exponent - 150;
		
		valueMarginLow = BigNumber:new{1};
		valueMarginHigh = BigNumber:new{1};
	--Normalized
	elseif exponent > 0 and exponent < 255 then
		--value = (1+mantissa/2^23) * (2 ^ (exponent-127));
		valueMantissa = 2^23 + mantissa;
		valueExponent = exponent - 150;
		--从非规格换浮点数到 exponent==1 的规格化浮点数，margin没有变化
		if exponent == 1 then
			valueMarginLow = BigNumber:new{1};
			valueMarginHigh = BigNumber:new{1};
		else
			--其他情况，当mantissa是0的时候，高margin是低margin的2倍大小
			if mantissa == 0 then
				valueMarginLow = BigNumber:new({1})
				valueMarginHigh = BigNumber:new({2})
			else
				valueMarginLow = BigNumber:new({1})
				valueMarginHigh = BigNumber:new({1})
			end
		end
	--Infinity
	elseif exponent == 255 and mantissa==0 then

	--NaN
	elseif exponent == 255 and mantissa~=0 then
	end
	value = (valueMantissa * 2^valueExponent);
	--表示成大整数除法
	local valueNumerator;
	local valueDenominator;

	if (valueExponent > 0) then
		valueNumerator = BigNumber:new({valueMantissa});
		shift_left(valueNumerator,valueExponent);
		shift_left(valueMarginLow,valueExponent);--高低margin都同样扩大
		shift_left(valueMarginHigh,valueExponent);
		valueDenominator = BigNumber:new({1});
	else
		valueNumerator = BigNumber:new({valueMantissa});
		valueDenominator = BigNumber:new({1});
		shift_left(valueDenominator,-valueExponent);
	end
	
	--都同时扩大2倍，这样就不用margin/2进行比较了
	shift_left(valueNumerator,1);
	shift_left(valueDenominator,1);
	do 
		--------------core---------------
		--这里不做优化了，只实现功能
		local zeroLeft,remainder = div(valueNumerator,valueDenominator);
		local TEN = BigNumber:new({10});
		local TWO = BigNumber:new({2});
		local ONE = BigNumber:new({1});
		local MINUS_ONE = BigNumber:new({-1});
		--print(getBigNumberDecimal(zeroLeft));
		local len = 40;
		local zeroRightNum;
		--在处理小数点左边数字的时候，不需要增加对margin乘以10
		--valueMarginLow = mult(valueMarginLow,TEN)
		--valueMarginHigh = mult(valueMarginHigh,TEN)
		local res_str = getBigNumberDecimal(zeroLeft) .. '.';
		while(len>0) do 
			valueNumerator = remainder;
			valueNumerator = mult(valueNumerator,TEN);
			zeroRightNum,remainder = div(valueNumerator,valueDenominator);
			
			valueMarginLow = mult(valueMarginLow,TEN)
			valueMarginHigh = mult(valueMarginHigh,TEN)

			--对比余数和两个margin
			if ( compare(remainder,valueMarginLow) == -1 or  compare( add(remainder,valueMarginHigh),valueDenominator) == 1 )then
				--判断remainder更接近lowMargin还是HighMargin，进行舍入
				--if (remainder - valueMarginLow < valueMarginHigh - remainder)--向下
				if( compare( mult(remainder,TWO) , add(valueMarginLow , valueMarginHigh ) ) == -1) then
					--zeroRightNum = zeroRightNum - 1;
					zeroRightNum = add(zeroRightNum ,MINUS_ONE);
				--if (remainder - valueMarginLow > valueMarginHigh - remainder)--向上
				elseif (compare( mult(remainder,TWO) , add(valueMarginLow , valueMarginHigh ) ) == 1) then
					--zeroRightNum = zeroRightNum + 1;
					zeroRightNum = add(zeroRightNum ,ONE);
				else--相等，如果remainder是奇数，向上取整，否则向下取整（IEEE做法）
					print("相等，未作处理")
				end
				--print(getBigNumberDecimal(zeroRightNum));
				res_str = res_str .. getBigNumberDecimal(zeroRightNum)
				break;
			end
			--print(getBigNumberDecimal(zeroRightNum));
			res_str = res_str .. getBigNumberDecimal(zeroRightNum)
			if remainder.int32s[1] == 0 then
				break;
			end
			len = len - 1;
		end
		
		print(res_str)
	end
end

--0x4123C28E = 01000001 00100011 11000010 10001110 10.2349987030029296875
--0x4123C28F = 01000001 00100011 11000010 10001111 10.23499965667724609375
--0x4123C290 = 01000001 00100011 11000010 10010000 10.2350006103515625
--把10.235转换成float:
--0x4123C28F = 01000001 00100011 11000010 10001111 10.235
--发现其二进制竟然和 0x4123C28F一样，和精确值10.23499965667724609375的表示一样！


local exponent = 130;
local mantissa = 2343566;
dragon4(exponent,mantissa);

local exponent = 130;
local mantissa = 2343567;
dragon4(exponent,mantissa);

local exponent = 130;
local mantissa = 2343568;
dragon4(exponent,mantissa);

local mantissa = 4788187;
local exponent = 128;
dragon4(exponent,mantissa);




